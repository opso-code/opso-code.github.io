---
layout: post
title: MySQL5.7.10安装和phpmyadmin配置
date: 2016-01-07
author: opso
tags:
- MySQL
- phpmyadmin
cover: "/images/mysql_logo.jpg"
---

Ubuntu14.04下`apt-get install`安装的MySQL版本是5.5，想要安装5.7，编译源码方式安装太麻烦，在官网找到了一个比较方便的安装方式，通过deb添加mysql的安装源，最新可安装的版本是5.7.10；记录下安装过程和`phpmyadmin`使用配置。

<!--more-->

[2016-03-29] 更新 [MariaDB 10.1.13 编译安装](../mariadb-install)

### 环境：ubuntu14.04

[MySQL官网下载地址](http://dev.mysql.com/downloads/mysql/)
首页找到`Download` ==> `APT Repository`

[mysql-apt-config_0.6.0-1_all.deb](http://dev.mysql.com/downloads/repo/apt/)
 
```bash
wget http://dev.mysql.com/get/mysql-apt-config_0.6.0-1_all.deb
sudo dpkg -i mysql-apt-config_0.6.0-1_all.deb
#发现原来是添加了mysql的安装源
sudo apt-get update
sudo apt-get install mysql-server
#然后开始下载安装，中间会让你设置密码
$ mysql -V
 mysql  Ver 14.14 Distrib 5.7.10, for Linux (x86_64) using  EditLine wrapper
```

然后下载 [phpmyadmin](https://www.phpmyadmin.net/)
修改根目录下数据库连接配置`config.inc.php`，如果根目录下不存在

```text
cp libraries/config.default.php config.inc.php
sudo vim config.inc.php
```

```php
<?php
/* Authentication type */
$cfg['Servers'][$i]['auth_type'] = 'config';
$cfg['Servers'][$i]['user'] = 'root';
$cfg['Servers'][$i]['password'] = '123456';
/* Server parameters */
$cfg['Servers'][$i]['host'] = '127.0.0.1';
$cfg['Servers'][$i]['connect_type'] = 'tcp';
$cfg['Servers'][$i]['compress'] = false;
/* Select mysql if your server does not have mysqli */
$cfg['Servers'][$i]['extension'] = 'mysqli';
$cfg['Servers'][$i]['AllowNoPassword'] = true;
```
这时候提示 `配置文件权限错误，不应任何用户都能修改!`

```php
<?php
$cfg['CheckConfigurationPermissions'] = false;//改为false
```
修改mysql配置文件`/etc/mysql/my.cnf`，修改为`uft8`

```php
<?php
[mysqld]
character_set_server = utf8
```

重启mysql，登录后查看编码

```sh
sudo service mysql restart
mysql> show variables like 'character_set%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

### 新建user

```sql
create user 'vagrant'@'%' identified by 'vagrant'; 
```

### 赋予权限
通配符`%`,可以让该用户从任意远程主机登陆

```sql
GRANT ALL PRIVILEGES ON *.* TO 'vagrant'@'%' IDENTIFIED BY 'vagrant' WITH GRANT OPTION;
flush privileges;
```
这时候发现依旧不能登陆
`#2002 服务器没有响应（或本地服务器的套接字没有正确设置）。` 

继续修改`config.inc.php`;

```php
<?php
// $cfg['Servers'][$i]['host'] = 'localhost';
//将localhost改为127.0.0.1 正常
$cfg['Servers'][$i]['host'] = '127.0.0.1';
//按照配置文件自动登陆，设置为cookie时需手动登陆
$cfg['Servers'][$i]['auth_type'] = 'config';
```

