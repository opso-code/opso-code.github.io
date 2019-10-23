---
layout: post
title: PHP7安装笔记(LNMP环境搭建)
date: 2016-01-20
author: opso
tags:
- php
- nginx
- lnmp
- sysv-rc-conf
cover: "/images/php_logo.jpg"
---

**PHP7** 一个划时代的大版本，性能比`PHP5.6`提升两倍，让无数PHPer奔走相告！
`php7.0`正式版出来也好久了，之前在灵雀云上试着装过，太慢，这次在本地的虚拟机里装，也没遇到什么坎坷，包括`php-fpm`、`nginx`配置，整理记录。

<!--more-->

![lnmp](/images/blog-lnmp.jpg)

# 替换中文源

环境：Ubuntu14.04（虚拟机）

首先更改ubuntu源为本地源，下载更新ubuntu更快 =、= 

```bash
sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
sudo vim /etc/apt/sources.list
```

添加中科院源和163网易源

```bash
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-security main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-updates main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-proposed main restricted universe multiverse
deb-src http://mirrors.ustc.edu.cn/ubuntu/ trusty-backports main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ vivid main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ vivid-security main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ vivid-updates main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ vivid-proposed main restricted universe multiverse
deb-src http://mirrors.163.com/ubuntu/ vivid-backports main restricted universe multiverse
```

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

更新源的时候如果遇到

```text
E: Could not get lock /var/lib/dpkg/lock - open (11: Resource temporarily unavailable)
E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?
```

解决办法： 删除锁，重新更新

```bash
sudo rm -rf /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock
sudo apt-get update
```

# 下载php源码

一种是使用git方式获取最新源码，这个方法下载的代码都是正在开发中，适合体验新版本功能；

```bash
sudo apt-get install git
mkdir $HOME/php7
cd $HOME/php7
git clone https://git.php.net/repository/php-src.git
```

另外一种是去php官网下载源码包

```bash
sudo wget http://cn2.php.net/distributions/php-7.0.1.tar.gz
sudo tar zxf php-7.0.1.tar.gz
cd php-7.0.1
```

# 安装必要组件命令

```bash
sudo apt-get update 

sudo apt-get install -y \
build-essential \
autoconf \
libtool \
re2c \
libxml2-dev \
openssl \
libcurl4-openssl-dev \
libssl-dev \
libbz2-dev \
libjpeg-dev \
libpng12-dev \
libfreetype6-dev \
libldap2-dev \
libmcrypt-dev \
libmysqlclient-dev \
libxslt1-dev \
libxt-dev \
libpcre3-dev \
libxpm-dev \
libt1-dev \
libgmp-dev \
libicu-dev \
libpspell-dev \
librecode-dev \
libreadline6-dev

#sudo ln -s /usr/lib/`arch`-linux-gnu/libldap.so /usr/lib/
#sudo ln -s /usr/lib/`arch`-linux-gnu/liblber.so /usr/lib/
sudo ln -s /usr/include/`arch`-linux-gnu/gmp.h /usr/include/gmp.h 
```
# 编译php

```bash
./buildconf
./configure \
--prefix=/usr/local/php7 \
--enable-bcmath \
--enable-calendar \
--enable-dba \
--enable-exif \
--enable-fpm \
--enable-ftp \
--enable-gd-jis-conv \
--enable-gd-native-ttf \
--enable-intl \
--enable-mbstring \
--enable-opcache \
--enable-pcntl \
--enable-pdo \
--enable-shmop \
--enable-soap \
--enable-sockets \
--enable-sysvmsg \
--enable-sysvsem \
--enable-sysvshm \
--enable-wddx \
--enable-zip \
# --with-apxs2=/usr/local/apache2/bin/apxs \
--with-bz2=/usr \
--with-curl \
--with-freetype-dir=/usr \
--with-gd \
--with-gettext=/usr \
--with-gmp \
--with-gmp=/usr \
--with-iconv \
--with-icu-dir=/usr \
--with-jpeg-dir=/usr \
--with-ldap \
--with-libdir=/lib/x86_64-linux-gnu \
--with-mcrypt \
--with-mhash \
--with-mysqli \
--with-openssl \
--with-pcre-regex \
--with-pdo-mysql \
--with-pdo-sqlite \
--with-pear \
--with-png-dir=/usr \
--with-pspell \
--with-readline \
--with-recode=/usr \
--with-xmlrpc \
--with-xpm-dir=/usr \
--with-xsl \
--with-zlib-dir=/usr \
--with-zlib=/usr \
--with-fpm-group=www \
--with-fpm-user=www
```

编译报错

> configure: WARNING: This bison version is not supported for regeneration of the Zend/PHP parsers

下载最新 `bison3.0.4`

```bash
wget http://mirror.hust.edu.cn/gnu/bison/bison-3.0.4.tar.gz
tar -zxf bison-3.0.4.tar.gz
cd bison-3.0.4
./configure
make && sudo make install
```

继续编译php7，无报错
`make && sudo make install`

```bash
Installing shared extensions:     /usr/local/php7/lib/php/extensions/no-debug-non-zts-20151012/
Installing PHP CLI binary:        /usr/local/php7/bin/
Installing PHP CLI man page:      /usr/local/php7/php/man/man1/
Installing PHP FPM binary:        /usr/local/php7/sbin/
Installing PHP FPM config:        /usr/local/php7/etc/
Installing PHP FPM man page:      /usr/local/php7/php/man/man8/
Installing PHP FPM status page:      /usr/local/php7/php/php/fpm/
Installing phpdbg binary:         /usr/local/php7/bin/
Installing phpdbg man page:       /usr/local/php7/php/man/man1/
Installing PHP CGI binary:        /usr/local/php7/bin/
Installing PHP CGI man page:      /usr/local/php7/php/man/man1/
Installing build environment:     /usr/local/php7/lib/php/build/
Installing header files:          /usr/local/php7/include/php/
Installing helper programs:       /usr/local/php7/bin/
  program: phpize
  program: php-config
Installing man pages:             /usr/local/php7/php/man/man1/
  page: phpize.1
  page: php-config.1
Installing PEAR environment:      /usr/local/php7/lib/php/
Wrote PEAR system config file at: /usr/local/php7/etc/pear.conf
You may want to add: /usr/local/php7/lib/php to your php.ini include_path
/home/vagrant/php7/php-src/build/shtool install -c ext/phar/phar.phar /usr/local/php7/bin
ln -s -f phar.phar /usr/local/php7/bin/phar
Installing PDO headers:          /usr/local/php7/include/php/ext/pdo/
```

# php配置文件

```bash
#php安装目录下
sudo cp php.ini-* /usr/local/php7/lib/
#sudo cp sapi/fpm/init.d.php-fpm /usr/local/php7/php-fpm
#想要执行service php-fpm start|stop 
sudo cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
sudo chmod +x /etc/init.d/php-fpm
#ubuntu里没有`chkconfig`命令，通过`apt-get install sysv-rc-conf` 替代
sudo sysv-rc-conf php-fpm
sudo sysv-rc-conf php-fpm on

sudo cp sapi/fpm/php-fpm.service /usr/local/php7/
sudo cp /usr/local/php7/lib/php.ini-production /usr/local/php7/lib/php.ini
sudo cp /usr/local/php7/etc/php-fpm.conf.default /usr/local/php7/etc/php-fpm.conf
sudo cp /usr/local/php7/etc/php-fpm.d/www.conf.default /usr/local/php7/etc/php-fpm.d/www.conf
```

这里的`www.conf`是php-fpm的配置，默认是用编译参数生成的`www`用户组和用户名，如果没有这个用户，我们需要先创建，否组php-fpm将不能运行

```bash
sudo groupadd www
sudo useradd www www
```

如果需要开启opcache，修改`php.ini`

```Ini
sudo vim /usr/local/php7/lib/php.ini
#搜索opcache
[opcache]
zend_extension= /usr/local/php7/lib/php/extensions/no-debug-non-zts-20151012/opcache.so
opcache.enable=1
opcache.enable_cli=1
```


# 安装Nginx

```bash
#Nginx官网提供了三个类型的版本
#Mainline version：Mainline 是 Nginx 目前主力在做的版本，可以说是开发版
#Stable version：最新稳定版，生产环境上建议使用的版本
#Legacy versions：遗留的老版本的稳定版
#nginx下载地址：http://nginx.org/en/download.html
#目前稳定版是1.8.0，开发版1.9.9
wget http://nginx.org/download/nginx-1.8.0.tar.gz
tar -zxf nginx-1.8.0.tar.gz
cd nginx-1.8.0/
./configure
  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/usr/local/nginx/logs/nginx.pid"
  nginx error log file: "/usr/local/nginx/logs/error.log"
  nginx http access log file: "/usr/local/nginx/logs/access.log"
  nginx http client request body temporary files: "client_body_temp"
  nginx http proxy temporary files: "proxy_temp"
  nginx http fastcgi temporary files: "fastcgi_temp"
  nginx http uwsgi temporary files: "uwsgi_temp"
  nginx http scgi temporary files: "scgi_temp"
make && sudo make install 
```

接下来需要修改`/usr/local/nginx/conf/nginx.conf`，有时候一台主机并不是只有一个域名或者站点，为了方便管理，可以将虚拟主机配置统一放到一起，在同一目录下新建`vhosts.conf`，我们需要在nginx.conf中添加下面一句引入

```bash
include vhosts.conf
```

之后就可以在vhosts.conf中加入对php解析

```nginx
server {
	listen       80;
	server_name  ubuntu.dev;
	root   /vagrant/www;
	location / {
		index  index.html index.htm index.php;
		#autoindex  on;
	}

	location ~ \.php(.*)$ {
		fastcgi_pass   127.0.0.1:9000;
		fastcgi_index  index.php;
		fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
		fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
		fastcgi_param  PATH_INFO  $fastcgi_path_info;
		fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
		include        fastcgi_params;
	}
}
```

# 启动Nginx

```bash
#制定记载配置（如果是在sbin目录下则不需要）
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
/usr/local/nginx/sbin/nginx -s reload
```

这样是不能通过 `service nginx start|stop` 来启动停止的，下面是个脚本nginx，放到 `/etc/ini.d/`目录下，然后执行

```bash
#网上下载一个nginx启动的脚本
#git https://github.com/JasonGiedymin/nginx-init-ubuntu.git
sudo wget http://dlj.bz/OHaQ -O /etc/init.d/nginx
sudo chmod +x /etc/init.d/nginx
#加入系统启动项
sudo sysv-rc-conf nginx
#开启开机启动
sudo sysv-rc-conf nginx on
```


# 其他

```bash
#创建网站根目录
mkdir /vagrant/www
#提升权限
sudo chmod 775 -R /vagrant/www
#或更改所属用户
sudo chown vagrant:vagrant -R /vagrant/www
```
ubuntu里没有`chkconfig`命令，通过`apt-get install sysv-rc-conf` 替代

```bash
sudo sysv-rc-conf php-fpm
sudo sysv-rc-conf php-fpm on
```
查看nginx日志的时候，发现Ubuntu的时区有问题，`date`命令返回的都是晚8小时，修改Ubuntu系统时区。

```bash
#按照提示修改时区
sudo tzselect
sudo cp /usr/share/zoneinfo/Asia/Shanghai  /etc/localtime

#或者 使用tzdata 视图中重新选择时区
sudo dpkg-reconfigure tzdata
```

# 资料
> 视频：[php7.0新特性](http://www.imooc.com/learn/438)
> 鸟哥的博客：[让PHP7达到最高性能的几个Tips](http://www.laruence.com/2015/12/04/3086.html)