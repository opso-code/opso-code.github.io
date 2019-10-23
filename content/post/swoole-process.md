---
layout: post
title: swoole_process实现进程池
date: 2018-07-07
author: opso
tags:
- swoole
- php
cover: "/images/swoole_logo.png"
---

**Swoole** —— 重新定义PHP

作为一个使用C语言底层实现的PHP扩展，弥补了PHP在多线程、常驻内存相关领域的短处，为广大phper打开了眼界。

`swoole` 的进程之间有两种通信方式，一种是`消息队列(queue)`，另一种是`管道(pipe)`，对`swoole_process` 的研究在swoole中显得尤为重要。

<!--more-->

## 预备知识

### IO多路复用

`swoole` 中的io多路复用表现为底层的 `epoll进程模型`，在C语言中表现为 `epoll` 函数。

- `epoll` 模型下会持续监听自己名下的素有`socket` 描述符 `fd`
- 当触发了 `socket` 监听的事件时，`epoll` 函数才会响应，并返回所有监听该时间的 `socket` 集合
- `epoll` 的本质是`阻塞IO`，它的优点在于能同事处理大量socket连接

### Event loop 事件循环

`swoole` 对 `epoll` 实现了一个`Reactor线程模型`封装，设置了`read事件`和`write事件`的监听回调函数。（详见`swoole_event_add`）
- `Event loop` 是一个`Reactor线程`，其中运行了一个`epoll`实例。
- 通过`swoole_event_add`将socket描述符的一个事件添加到epoll监听中，事件发生时将执行回调函数
- 不可用于`fpm`环境下，因为`fpm`在任务结束时可能会关掉进程。

## swoole_process

- 基于C语言封装的进程管理模块，方便php来调用
- 内置管道、消息队列接口，方便实现进程间通信

我们在`php-fpm.conf`配置文件中发现，`php-fpm`中有两种进程池管理设置。

- 静态模式 即初始化固定的进程数，当来了一个请求时，从中选取一个进程来处理。
- 动态模式 指定最小、最大进程数，当请求量过大，进程数不超过最大限制时，新增线程去处理请求

接下来用swoole代码来实现，这里只是为理解`swoole_process`、进程间通信、定时器等使用，实际情况使用封装好的`swoole_server`来实现`task`任务队列池会更方便。

假如有个定时投递的任务队列：

```php
<?php

/**
 * 动态进程池，类似fpm
 * 动态新建进程
 * 有初始进程数，最小进程数，进程不够处理时候新建进程，不超过最大进程数
 */

// 一个进程定时投递任务

/**
 * 1. tick
 * 2. process及其管道通讯
 * 3. event loop 事件循环
 */
class processPool
{
    private $pool;

    /**
     * @var swoole_process[] 记录所有worker的process对象
     */
    private $workers = [];

    /**
     * @var array 记录worker工作状态
     */
    private $used_workers = [];

    /**
     * @var int 最小进程数
     */
    private $min_woker_num = 5;

    /**
     * @var int 初始进程数
     */
    private $start_worker_num = 10;

    /**
     * @var int 最大进程数
     */
    private $max_woker_num = 20;

    /**
     * 进程闲置销毁秒数
     * @var int
     */
    private $idle_seconds = 5;

    /**
     * @var int 当前进程数
     */
    private $curr_num;

    /**
     * 闲置进程时间戳
     * @var array
     */
    private $active_time = [];

    public function __construct()
    {
        $this->pool = new swoole_process(function () {
            // 循环建立worker进程
            for ($i = 0; $i < $this->start_worker_num; $i++) {
                $this->createWorker();
            }
            echo '初始化进程数：' . $this->curr_num . PHP_EOL;
            // 每秒定时往闲置的worker的管道中投递任务
            swoole_timer_tick(1000, function ($timer_id) {
                static $count = 0;
                $count++;
                $need_create = true;
                foreach ($this->used_workers as $pid => $used) {
                    if ($used == 0) {
                        $need_create = false;
                        $this->workers[$pid]->write($count . ' job');
                        // 标记使用中
                        $this->used_workers[$pid] = 1;
                        $this->active_time[$pid] = time();
                        break;
                    }
                }
                foreach ($this->used_workers as $pid => $used)
                    // 如果所有worker队列都没有闲置的，则新建一个worker来处理
                    if ($need_create && $this->curr_num < $this->max_woker_num) {
                        $new_pid = $this->createWorker();
                        $this->workers[$new_pid]->write($count . ' job');
                        $this->used_workers[$new_pid] = 1;
                        $this->active_time[$new_pid] = time();
                    }

                // 闲置超过一段时间则销毁进程
                foreach ($this->active_time as $pid => $timestamp) {
                    if ((time() - $timestamp) > $this->idle_seconds && $this->curr_num > $this->min_woker_num) {
                        // 销毁该进程
                        if (isset($this->workers[$pid]) && $this->workers[$pid] instanceof swoole_process) {
                            $this->workers[$pid]->write('exit');
                            unset($this->workers[$pid]);
                            $this->curr_num = count($this->workers);
                            unset($this->used_workers[$pid]);
                            unset($this->active_time[$pid]);
                            echo "{$pid} destroyed\n";
                            break;
                        }
                    }
                }

                echo "任务{$count}/{$this->curr_num}\n";

                if ($count == 20) {
                    foreach ($this->workers as $pid => $worker) {
                        $worker->write('exit');
                    }
                    // 关闭定时器
                    swoole_timer_clear($timer_id);
                    // 退出进程池
                    $this->pool->exit(0);
                    exit();
                }
            });

        });

        $master_pid = $this->pool->start();
        echo "Master $master_pid start\n";

        while ($ret = swoole_process::wait()) {
            $pid = $ret['pid'];
            echo "process {$pid} existed\n";
        }
    }

    /**
     * 创建一个新进程
     * @return int 新进程的pid
     */
    public function createWorker()
    {
        $worker_process = new swoole_process(function (swoole_process $worker) {
            // 给子进程管道绑定事件
            swoole_event_add($worker->pipe, function ($pipe) use ($worker) {
                $data = trim($worker->read());
                if ($data == 'exit') {
                    $worker->exit(0);
                    exit();
                }
                echo "{$worker->pid} 正在处理 {$data}\n";
                sleep(5);
                // 返回结果，表示空闲
                $worker->write("complete");
            });
        });

        $worker_pid = $worker_process->start();

        // 给父进程管道绑定事件
        swoole_event_add($worker_process->pipe, function ($pipe) use ($worker_process) {
            $data = trim($worker_process->read());
            if ($data == 'complete') {
                // 标记为空闲
//                echo "{$worker_process->pid} 空闲了\n";
                $this->used_workers[$worker_process->pid] = 0;
            }
        });

        // 保存process对象
        $this->workers[$worker_pid] = $worker_process;
        // 标记为空闲
        $this->used_workers[$worker_pid] = 0;
        $this->active_time[$worker_pid] = time();
        $this->curr_num = count($this->workers);
        return $worker_pid;
    }

}

new processPool();
```