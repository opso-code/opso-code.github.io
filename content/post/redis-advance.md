---
layout: post
author: opso
title: Redis常用命令和高级特性
date: 2015-10-20
tags:
- redis
---

Redis常用命令和高级特性

<!--more-->

### 安装Redis
`sudo apt-get install redis-server`

查看redis进程
`ps -aux|grep redis`

查看端口6379是否被占用
`netstat –ntlp |grep 6379`

修改redis配置文件
`vim /etc/redis/redis.conf `

查看redis版本
`redis-server -v`

redis启动/停止/重启/强制重启/运行状态
`redis-server {start|stop|restart|force-reload|status}`

----

### Redis键值相关命令

1.	 `exists key`  判断键值是否存在
	 返回 `int` 类型 `1`表示存在 ，`0`不存在

2.	 `keys`+ `通配符` 查询符合条件的键
	`?` 匹配一个字符
	`*` 匹配所有键
	`[]` 范围内任意字符
	`\x`匹配字符x，用于转义

3.	 `del key` 删除键
	删除所有以`stu:`开头的键，利用了linux管道和`xargs`命令
	`redis-cli keys "stu:*" | xargs redis-cli del`
4.	`expire key seconds` 设置键的生存时间
5.	`persist key` 取消键的生存时间，即永久	

	redis有0-15，共16个数据库
6.	`remove key 0-15` 将键移动到其他数据库
7.	`randomkey` 随机获取key
8. 	`rename` 重命名
9. 	`type key`
	 返回数据的类型(`string` `list` `set` `zset` `hash`)
	
### Redis服务器相关命令
1.	`ping` 测试连接是否存活，返回`PONG`表示成功：）
2.	`echo` 输出一些内容，（类似于 ）	
3.	`quit` 退出连接
4.	`select db` 0-15选择数据库
5.	`dbsize`  返回当前数据库键的数量
6.	`info` 获取服务器的信息和统计
7.	`config get param` 获取服务器运行时参数 
8.	`flashdb`  清除当前数据库里所有键
9.	`flashall` 清除所有数据库里的键

### Redis高级特性-事务
1.	事务的处理

```shell
 127.0.0.1:6379>mult
 127.0.0.1:6379>sadd "user:001:following" 2
 QUEUED
 127.0.0.1:6379>sadd "user:2:following" 001
 QUEUED
 127.0.0.1:6379>exec
 1) integer 1
 2) integer 1
```

- 事务开始
- 命令入队
- 事务执行
`WATCH` 命令用于在事务开始钱监视人鱼数量的键：当调用`EXEC`命令执行事务是，如果任意一个被监视的键已经被其他客户端修改了，那么整个事务不再执行，直接返回失败

### Redis SORT排序

`SORT`排序 
ALPHA参数
`sort fruit alpha desc`  按字母倒序排列

BY参数
`mset apple-price 8 banana-price 5.5 cherry-price 7`
`sort fruits by *->price` 按照水果价格排序
`sort blogs by log:*->time` 按照博客发表时间排序

GET参数
`sort blogs by log:*->time get blog:*->title` 按照博客发表时间和标题排序
LIMIT选项（类似于分页）

STORE参数
`sort fruits alpha store sorted_fruits`
将排序结果保存到一个list中

- 排序
- 限制排序长度
- 获取外部键
- 保存排序结果集
- 向客户端返回排序结果集
- 除GET选项外其他选项摆放顺序改变不影响结果

