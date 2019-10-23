---
layout: post
title: PHP常见扩展功能说明
date: 2017-03-31
author: opso
tags:
- php
cover: "/images/php_logo.jpg"
---

每次编译安装PHP的时候都会有一堆扩展参数，有些看名字就知道其中的作用，有些不甚了解，有时间查看了下官方文档。

<!--more-->

官方文档都很详细，了解了一些扩展的功能，在编译安装的时候就可以选择性的安装和卸载，提升服务器性能完善功能。

[PHP手册-扩展列表](http://php.net/manual/zh/extensions.php)

常见扩展说明：

- 数据库扩展
	- PDO 数据库抽象层
	- pdo_mysql mysql PDO扩展
	- PDO_ODBC php ODBC驱动扩展
	- pdo_sqlite SQLLite扩展
	- sqlite3
	- mysqli 数据库扩展
	- mysqlnd 数据库扩展
	- frontbase 一种数据库 php5.3后不再捆绑
	- phpredis Redis数据库扩展
	- memcached Memcache数据库扩展
- 图像处理
	- imagemagick 图像处理 (验证码等)
	- gd 图像处理
- 字符编码、国际化
	- gettext 用于国际化
	- mbstring 多字节字符串处理 中文字符长度问题
	- pcre 正则perl
	- iconv 字符集转换功能
	- recode 字符编码
- 数学计算
	- bcmath 高精度的数字计算
	- GMP 数学扩展
- 压缩与归档扩展
	- bz2 .gz文件
	- zip .zip文件
	- zlib
- 基本类库扩展
    - Core 核心类
    - SPL PHP标准库 
    - Standard PHP标准库
    - Sessions
    - JSON JSON格式支持 json_decode等
    - XML xml格式支持
    - Tidy HTML整理修复工具
    - Tokenizer Zend引擎解析器
    - Yaml YAML 数据序列化
    - Ctype 字符类型检测
    - FTP
    - XML-RPC xml远程过程调用
    - Sockets socket通讯
    - curl 数据连接通讯
    - Date 日期时间
    - Hash 信息摘要（哈希）引擎
    - Phar 用来将多个PHP文件打包为一个文件
    - Reflection 反射
    - POSIX 进程控制扩展
    - pthreads 多进程实现类
    - Zend OPcache php字节码缓存
- XML扩展
    - Dom
    - libxml
    - SimpleXML
    - wddx Web分布式数据交换(xml)
    - xmlreader
    - xmlwriter
    - xsl
- 其他功能辅助性扩展
    - Calendar 日期时间扩展
    - IMAP email 服务相关
    - LDAP 轻量级目录访问服务
    - Pspell 拼写检查
    - Readline 脚本命令行工具
    - SNMP 网络相关 Simple Network Management Protocol. 
    - SOAP WebService的一种
- 数据加密、过滤扩展
    - filter 数据过滤 如email格式检测 FILTER_VALIDATE_EMAIL
    - mcrypt 加密扩展
    - mhash 加密相关
    - OpenSSL 加密扩展
