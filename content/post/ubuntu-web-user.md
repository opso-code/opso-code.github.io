---
layout: post
title: Composer命令root用户权限问题
date: 2017-02-01
author: opso
tags:
- linux
- www
cover: "/images/linux_logo.jpg"
---

**Composer** 作为php的包管理工具，今天在运行`composer`命令的时候，遇到了 `Do not run Composer as root/super user!` 权限提示问题，才知道是不能在root用户下运行的。

为了统一`php-fpm`、`nginx`、`web`目录三者权限，今天有空查了一些资料，实际操作了一遍。

<!--more-->

在运行`composer`命令的时候:

> php composer.phar install

的时候会提示没有权限访问xxx目录，然而加上`sudo`管理员权限运行的时候会导致生成的文件夹权限为root，这就很头疼了。

访问官方网站，给出的原因是出于安全考虑，防止第三方组件以管理员权限运行代码，挺有道理的，于是就很纠结了。

> * 环境：ubuntu 16.04
> * 当前用户：opso
> * Web服务器：编译安装，默认用户`nobody`
> * PHP：apt安装，默认用户`www-data`
> * Web目录：`/data/www`，用户/用户组 `root:root`

之前也一直觉得给Web目录设置root权限并不安全，每次修改代码都要加上可恶的 `sudo`，

先查看php-fpm和Nginx运行的用户

```bash
$ ps -ef | grep nginx
$ ps -ef | grep php-fpm
```

可以判断出nginx和php-fpm主进程都是root权限启动（加入了自启动和`sudo service xxx start`），nginx子进程用的 `nobody` 用户下运行，php-fpm子进程则是`www-data`用户下运行，所以这里统一用户权限为`www-data`用户

修改`php-fpm`配置`/usr/local/nginx/conf/nginx.conf`如下，保存后重启nginx，访问正常。

```nginx
#user  nobody;
user  www-data;
worker_processes  1;
```

接下来修改web目录权限，

```bash
$ sudo chown -R www-data:www-data /data/www
```

当前用户(opso)依旧不能编辑文件于是尝试:

```bash
$ sudo chown -R opso:www-data /data/www
```

从此我的用户修改代码不用非得加`sudo`了，`Composer` 命令也正常了！:)
