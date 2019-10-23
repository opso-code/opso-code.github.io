---
layout: post
title: PHP交互式控制台——Psysh
date: 2018-08-17
author: opso
tags: 
 - php
 - composer
 - psysh
cover: "/images/php_logo.jpg"
---

今天来介绍一个PHP新的玩具：**PsySH** : http://psysh.org/

<!--more-->

`PsySH` 是基于php的可交互式控制台。他可以：

- 运行php命令
- 查看php文档
- tab补全
- 帮助调试php
- 反射列表

可以帮他当做一个php文档查询命令行，也可以当做一个php debug工具

## 安装

1. 安装方式有多种：

```sh
$ wget https://git.io/psysh
$ chmod +x psysh
$ ./psysh
```

2. `composer` 安装:

先安装 `composer` :

```bash
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php composer-setup.php --filename=composer
$ php -r "unlink('composer-setup.php');"
$ sudo mv composer /usr/local/bin
$ sudo chown root:root /usr/local/bin/composer
$ sudo chmod 755 /usr/local/bin/composer
```

再安装 `psysh`:

```bash
$ composer config -g repositories.packagist composer https://packagist.phpcomposer.com
$ composer g require psy/psysh:@stable
$ echo "PATH=$PATH:$HOME/.config/composer/vendor/bin" | tee -a ~/.profile
$ source ~/.profile
$ psysh
```

## Doc文档

> doc 方法名

默认情况下只有英文的说明，我们可以去下载 [php_manual.sqlite](http://psysh.org/manual/zh/php_manual.sqlite) 简体中文的详细说明的SQL数据库文件

```sh
$ mkdir ~/.local/share/psysh
$ cd ~/.local/share/psysh
$ wget http://psysh.org/manual/zh/php_manual.sqlite
```

![psysh](/images/psysh-bash.png)

## 引入框架

在代码中引入 `composer` 的自动加载

```php
<?php
\Psy\Shell::debug(get_defined_var());
```

使用内置的web服务器启动php：

```sh
$ php -S localhost:8080
```

psysh 就会在命令行打开了~ 可以运行框架里的方法了！