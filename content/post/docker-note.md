---
layout:	post
title: Docker安装笔记
date: 2015-12-26
author: opso
tags:
- docker
cover: "/images/docker_logo.png"
---

*环境：Ubuntu14.04*

 **Docker** 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 `Linux` 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口(类似 iPhone 的 app)。几乎没有性能开销,可以很容易地在机器和数据中心中运行。最重要的是,他们不依赖于任何语言、框架包括系统。
 
<!--more-->

#### 一、安装Docker

在ubuntu下可以用
`apt-get install docker.io`来安装docker，但是版本过低，官方有简单的安装方式

```bash
sudo apt-get update
sudo apt-get -y upgrade
curl -sSL https://get.docker.com/ | sudo sh
```

这里安装终止了，查了下是`apparmor`没有安装更新
`sudo apt-get install apparmor`

中间还会由于ubuntu 14.04的内核版本过低，上面的脚本会自动升级ubuntu内核到`3.13.0`，内核升级完后会接着安装最新的`docker 1.9.1`

安装成功后再执行上面的`docker`安装

```bash
$ docker --version
Docker version 1.9.1, build a34a1d5
```

#### 二、运行Docker

1. 新建`index.php`，里面有个`phpinfo()`；

2. 新建`Dockerfile`

3. 构建Docker php镜像

```bash
FROM daocloud.io/php:5.6-cli
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD [ "php", "./index.php" ]
```


```bash
docker build -t php_test .
docker run -it --rm --name test1 php_test
```

