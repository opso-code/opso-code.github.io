---
layout: post
title: Redis3.0安装和配置
date: 2016-04-07 18:05:49
tags: 
- reids
- php
---

**Redis** 是一个开源的C语言编写的`key-value`类型的`NoSQL`数据库。由于redis数据库存储在内存中，所以执行速度非常快，同样还有数据持久化，可以将数据备份移动到其他从服务器。Redis的消息队列（原生支持发布/订阅）,能稳定快速的处理高并发下的秒杀或者抢红包之类的需求，在游戏中的排行榜功能也是简单快速了许多。

<!--more-->

## 下载

```bash
$ wget http://download.redis.io/releases/redis-3.0.7.tar.gz
$ cd redis-3.0.7
$ make
$ cd src && make test
make[1]: Entering directory '/home/vagrant/src/redis-3.0.7/src'
You need tcl 8.5 or newer in order to run the Redis test
make[1]: *** [test] Error 1
make[1]: Leaving directory '/home/vagrant/src/redis-3.0.7/src'
make: *** [test] Error 2
```

官方建议在make之后执行`make test`，不过需要安装`tcl`，后来才知道也可以不装。

```bash
$ sudo apt-get install tcl
```

## 安装

```bash
$ sudo make PREFIX=/usr/local/redis install
```

安装完后，在`/usr/local/redis/bin`下生成了

* redis-benchmark
* redis-check-dump
* redis-sentinel
* redis-check-aof
* redis-cli
* redis-server

几个文件，整个安装目录也很小。

```bash
$ sudo cp redis.conf /usr/local/redis/6379.conf
$ sudo cp utils/redis_init_script /etc/init.d/redis
```

`redis_init_script`是redis自带的启动脚本，我们可以用sysv-rc-conf命令把redis加入到自启动中，也可以快速启动/停止，修改成我们需要的，启动默认配置文件是`/usr/local/redis/端口号.conf`，pid文件路径为`var/run/redis_端口号.pid`。

```sh
#!/bin/sh
REDISPORT=6379
EXEC=/usr/local/redis/bin/redis-server
CLIEXEC=/usr/local/redis/bin/redis-cli

PIDFILE=/var/run/redis_${REDISPORT}.pid
CONF="/usr/local/redis/${REDISPORT}.conf"
case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
                echo "$PIDFILE exists, process is already running or crashed"
        else
                echo "Starting Redis server..."
                $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
                echo "$PIDFILE does not exist, process is not running"
        else
                PID=$(cat $PIDFILE)
                echo "Stopping ..."
                $CLIEXEC -p $REDISPORT shutdown
                while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown ..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    *)
        echo "Please use start or stop as first argument"
        ;;
esac
```

## 配置

```bash
$ sudo mkdir -p /data/redis/logs
$ sudo mkdir -p /data/redis/data
```

修改`6379.conf`

```conf
logfile "/data/redis/logs/redis_6379.log"
dir ./data/redis/data
appendonly no
requirepass password
```


## phpredis扩展安装
**phpredis** 是官方推荐的连接redis的扩展，不过正式版本不支持php7，在开发者的github上php7分支已经支持了php7。

```bash
$ git clone https://github.com/phpredis/phpredis -b php7
$ cd phpredis
$ phpize
$ ./configure
$ make && sudo make install
```

`php.ini`加上

```ini
[redis]
extension=redis.so
```

## 启动

```bash
$ sudo service redis start
```

随机插入100条数据，如果不存在则创建，存在则跳过

```php
<?php
$redis = new Redis();
$redis->connect('127.0.0.1',6379);
for ($i=0; $i < 100; $i++) { 
    $key = 'test'.$i;
    $redis->setnx($key,rand(1,1000));
}
```

Bingo!

另外推荐window下一款redis可视化管理工具：[Redis Desktop Manager](http://redisdesktop.com/download)