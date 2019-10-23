---
title: Apache2.4 编译安装
date: 2016-03-30 10:23:24
tags:
- apache
- nginx
---

**Apache HTTP Server**(简称Apache)，由于其跨平台和安全性，被广泛使用，是最流行的Web服务器软件之一。
另一个强大快速的开源的web服务器 **Nginx**（读作engine x）常被人拿来与它比较，术业有专攻，下面是它们的一些对比：

<!--more-->

Apache 相对 Nginx 的优点：
* 动态资源处理优势
* 模块扩展多，全面
* 稳定性比nginx好
* PHP支持配置相对简单
* rewrite重写功能比nginx强大，配置`.htaccess`文件实现

Nginx 相对 Apache 的优点：
* 轻量级，低消耗，静态资源处理优势
* 抗并发，异步处理，Apache则是同步多进程模型
* 模块化设计，编写模块相对简单
* 可做反向代理服务器，支持负载均衡

说了那么多，存在即合理。之前熟悉了Nginx的安装配置，下面是Apache的编译安装

![httpd_logo](/images/httpd_logo.png)
## 开始安装

Apache <http://mirrors.cnnic.cn/apache//httpd/httpd-2.4.18.tar.gz>
apr <http://mirrors.cnnic.cn/apache//apr/apr--1.5.2.tar.gz>
apr-util <http://mirrors.cnnic.cn/apache//apr/apr-util-1.5.4.tar.gz>
pcre <http://iweb.dl.sourceforge.net/project/pcre/pcre/8.38/pcre-8.38.tar.gz>

安装Apache必须安装`apr`和`apr-util`;为了方便，我写了一个脚本 `install.sh`

```sh
#!bin/bash
tar zxf apr-1.5.2.tar.gz
tar zxf apr-util-1.5.4.tar.gz
tar zxf  pcre-8.37.tar.gz
tar zxf  httpd-2.4.18.tar.gz
cd apr-1.5.2
./configure --prefix=/usr/local/apr
make && sudo make install
cd ../apr-util-1.5.4
./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
make && sudo make install
cd ../pcre-8.37
./configure --prefix=/usr/local/pcre
make && sudo make install
cd ../httpd-2.4.18
./configure  --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-pcre=/usr/local/pcre
make && sudo make install
```
## 配置
修改`/usr/local/apache2/conf/httpd.conf`配置文件
部分配置：

```apache
ServerName localhost
#修改代码目录
DocumentRoot  "/vagrant/www"
<Directory "/vagrant/www">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
    Require all granted
</Directory>
#增加默认解析index文件
<IfModule dir_module>
    DirectoryIndex index.html index.php index.htm
</IfModule>
#增加php文件解析
AddType application/x-httpd-php .php
#引入虚拟目录配置文件
Include conf/vhosts.conf
```

## 启动apache
```bash
$ sudo /usr/local/apache2/bin/apachectl start
```

## PHP解析
apache是以插件形式解析php文件，也就是需要编译出`libphp7.so`给apache加载。

```bash
cd httpd-2.4.18
/usr/local/php7/bin/phpize
./configure –with-php-config=/usr/local/php7/bin/php-config
make && sudo make install
```
然后我们检查下`httpd.conf`里的php模块是否已经加载。

```text
$ sudo /usr/local/apache2/bin/apachectl -M
Loaded Modules:
 core_module (static)
 so_module (static)
 http_module (static)
 mpm_event_module (static)
 authn_file_module (shared)
 authn_core_module (shared)
 authz_host_module (shared)
 authz_groupfile_module (shared)
 authz_user_module (shared)
 authz_core_module (shared)
 access_compat_module (shared)
 auth_basic_module (shared)
 reqtimeout_module (shared)
 filter_module (shared)
 mime_module (shared)
 log_config_module (shared)
 env_module (shared)
 headers_module (shared)
 setenvif_module (shared)
 version_module (shared)
 unixd_module (shared)
 status_module (shared)
 autoindex_module (shared)
 dir_module (shared)
 alias_module (shared)
 php7_module (shared)
```
或者直接查看httpd.conf

```
LoadModule php7_module        modules/libphp7.so
```
时候就可以启动php了。

```bash
$ sudo service php-fmp start
```

如果遇到 **403 Forbidden**，一般有几个原因：

* DocumentRoot 权限不够
* 目录下没有 index.php 文件
* 用户组不对
* 去掉 Deny from all，改为Allow from all
* 禁用 selinux 试试

![phpinfo](/images/php7_httpd.png)

