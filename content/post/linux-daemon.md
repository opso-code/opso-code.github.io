---
layout: post
title: Linux创建守护进程自启动脚本
date: 2018-05-07
author: opso
tags: [linux, deamon, swoole]
cover: "/images/linux_logo.jpg"
---

在开发过程中，我们一般都是使用 `ssh` 登录linux，在命令行下直接使用 `php-cli` 模式运行 `swoole`，一旦退出`ssh` 或者使用 `ctrl`+`c`，终端会收到 **HUP**（hangup）信号从而关闭其所有子进程 ,`swoole` 进程会自动关闭。

那有问题来了，没有靠谱的、可以让脚本在后台长期运行呢，答案是有的 :)

<!--more-->

首先，我有个`swoole` http服务需要后台运行在服务器上，绑定端口`9501`，查询资料了解到下面几种以守护进程方式运行脚本的方法，可能不太全面，欢迎补充 : )

PHP代码如下：

```php
<?php
/**
 * 使用Swoole实现http服务简单示例
 * User: WilliamYao
 * Date: 2018/4/28 16:18
 */
$http = new swoole_http_server("0.0.0.0", 9501);
$http->set(array(
	'reactor_num' => 2, // 主进程内事件处理线程的数量 一般设置为CPU核数的1-4倍
	'worker_num' => 4, // 启动的worker进程数。
));
// 计数器
$automic = new swoole_atomic(1);

$http->on('request', function (swoole_http_request $request, swoole_http_response $response) {
	global $automic;
	$automic->add(1); // 计数+1
	$response->header("Content-Type", "text/html; charset=utf-8");
	$response->end("<h2>FROM {$request->fd}: Hello Swoole. <br/> COUNT: #{$automic->get()} <br/>TIME: ".microtime(true)."</h2>");
});
$http->start();
```

## 一、nohub

最简单的后台运行脚本命令，`nohup` 能通过忽略 **HUP** 信号来使我们的进程避免中途被中断

```bash
$ nohub /usr/local/php/bin/php /data/www/swoole/echo/server.php > /dev/null 2>&1 &
```

## 二、/etc/ini.d/xxx

Debian发行版中，可以使用  `start-stop-daemon --start/--stop` 创建/停止后台进程
详细可以使用 `man start-stop-daemon` 查看帮助
主要参数：

- `--backgroud` 标识后台运行
- `--pidfile` 指定pid文件
- `--make-pidfile` 如果脚本程序本身不产生pid文件，此选项可以生成pid文件
- `--exec` 执行的脚本程序 后面接 `--` 可添加参数
- `--chuid` 指定运行用户/用户组

```sh
#! /bin/sh
 
### BEGIN INIT INFO
# Provides:          swoole-echo
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-NAMEription: starts the swoole-echo server
# Description:       starts swoole-echo using start-stop-daemon
### END INIT INFO
 
PATH=/usr/local/php/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

NAME=swoole-echo
PIDFILE=/var/run/$NAME.pid
DAEMON=/usr/local/php/bin/php
DAEMON_OPTS=/data/www/swoole/echo/server.php
USER=www
GROUP=www

START_OPTS="--start --background --make-pidfile --chuid ${USER}:${GROUP} --pidfile ${PIDFILE} --exec ${DAEMON} ${DAEMON_OPTS}"
STOP_OPTS="--stop --pidfile ${PIDFILE}"
 
test -x $DAEMON || exit 0
 
# 若指令传回值不等于0，则立即退出shell。 
set -e

start_method(){
	echo -n "Starting $NAME"
	start-stop-daemon $START_OPTS
	echo " done"
}

stop_method(){
	echo -n "Stopping $NAME"
        
	if [ ! -r $PIDFILE ] ; then
		echo "Warning, no pid file found - $NAME is not running ?"
		exit 1
	fi
	
	start-stop-daemon $STOP_OPTS
	sleep 1
	if [ -e $PIDFILE ]
		then rm $PIDFILE
	fi
	
	echo " done"
}

case "$1" in
  start)
        start_method
		;;
  stop)
        stop_method
        ;;
  restart)
        stop_method
        start_method
        ;;
  *)
        echo "Usage: $0 {start|stop|restart}" >&2
        exit 1
        ;;
esac
 
exit 0

```

加入启动项：
```bash
# 添加
$ sudo update-rc.d swoole-echo defaults
# 删除
$ sudo update-rc.d -f swoole-echo remove
```

Red Hat发行版中，可使用 `/etc/rc.d/init.d/functions` 脚本中定义的方法
常用方法：

- `daemon` : 启动某个服务。`/etc/init.d` 目录部分脚本的start使用到这个
- `killproc` : 杀死某个进程。`/etc/init.d` 目录部分脚本的stop使用到这个
- `pidfileofproc` : 寻找某个进程的pid 
- `pidofproc` : 类似上面的，只是还查找了pidof命令
- `status` : 返回一个服务的状态 

以上方法帮助用户编写启动脚本，再放到 `/etc/ini.d/` 目录下，Debian使用 `update-rc.d xxx defaults`，RedHat中使用 `chkconfig --add xxx` 来加载脚本，之后便可使用 `service` 命令来 `start/stop/restart/status` 管理服务了。

## 三、Supervisor

> Supervisor 是基于 Python 的进程管理工具，可以帮助我们更简单的启动、重启和停止服务器上的后台进程，是 Linux 服务器管理的效率工具。

Supervisor 有两个主要的组成部分：

- `supervisord`，运行 `Supervisor` 时会启动一个进程 `supervisord`，它负责启动所管理的进程，并将所管理的进程作为自己的子进程来启动，而且可以在所管理的进程出现崩溃时自动重启。
- `supervisorctl`，是命令行管理工具，可以用来执行 `stop、start、restart` 等命令，来对这些子进程进行管理。

只需要修改配置文件（就是这么简单。。）

```bash
$ vim supervisord.conf
[program:swoole-echo]
command=/usr/local/php/bin/php /data/www/swoole/echo/server.php;
```

接下来就可以使用管理进程了

```bash
$ supervisorctl stop swoole-echo // 开启进程
$ supervisorctl start swoole-echo // 关闭进程
```

## 四、Systemd

> systemd 是一个 Linux 系统基础组件的集合，提供了一个系统和服务管理器，运行为 PID 1 并负责启动其它程序。功能包括：支持并行化任务；同时采用 socket 式与 D-Bus 总线式激活服务；按需启动守护进程（daemon）；利用 Linux 的 cgroups 监视进程；支持快照和系统恢复；维护挂载点和自动挂载点；各服务间基于依赖关系进行精密控制。systemd 支持 SysV 和 LSB 初始脚本，可以替代 sysvinit。除此之外，功能还包括日志进程、控制基础系统配置，维护登陆用户列表以及系统账户、运行时目录和设置，可以运行容器和虚拟机，可以简单的管理网络配置、网络时间同步、日志转发和名称解析等。

`Systemd` 并不是一个命令，而是一组命令; 与`supervisorctl` 类似，有 `systemctl` 命令可实现 `sudo systemctl start xxx`命令，方便创建守护进程，启动和停止服务。

如果你的系统是 **Ubuntu15.04+/CentOS7+** `Systemd` 已经默认代替了 `init` 启动方式，对与第一、第二种后台启动方式，Systemd有更强大的命令参数，更简便的配置文件，不需要额外安装软件包，还支持自动拉起服务，可以说是非常强大了，强烈推荐~

1. 建立 `swoole-echo.service` 文件

```sh
[Unit]
Description=Echo Http Server
After=network.target
After=syslog.target

[Service]
Type=simple
LimitNOFILE=65535
ExecStart=/usr/local/php/bin/php /data/www/swoole/echo/server.php 
ExecReload=/bin/kill -USR1 $MAINPID
Restart=always

[Install]
WantedBy=multi-user.target graphical.target
```

2. 复制到 `/etc/systemd/system/` ，赋予755权限

```bash
# 创建软连接，加入守护进程列表
$ sudo systemd enable swoole-echo.service
# 重载守护进程配置
$ sudo systemctl daemon-reload
# 加入自启动
$ sudo systemctl enable swoole-echo.service
# 启动
$ sudo systemctl start swoole-echo.service
# 查看状态
$ sudo systemctl status swoole-echo.service
● swoole-echo.service - Echo Http Server By Swoole
   Loaded: loaded (/etc/systemd/system/swoole-echo.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2018-05-07 15:08:42 CST; 1s ago
 Main PID: 22385 (php)
    Tasks: 8
   Memory: 41.1M
      CPU: 36ms
   CGroup: /system.slice/swoole-echo.service
           ├─22385 /usr/local/php/bin/php /data/wwwroot/swoole/echo/server.php
           ├─22392 /usr/local/php/bin/php /data/wwwroot/swoole/echo/server.php
           ├─22395 swoole: worker process: 22385
           ├─22396 swoole: worker process: 22385
           ├─22397 swoole: worker process: 22385
           └─22398 swoole: worker process: 22385
```

这时，即使使用`Systemd` 创建的守护进程检测到异常会自动拉起服务！
比如 `kill 22385`，发现守护进程会自动重新拉起，简直完美！

## 参考

[阮一峰-Systemd 入门教程：命令篇](http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)