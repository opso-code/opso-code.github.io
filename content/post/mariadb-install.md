---
title: MariaDB 10.1.13编译安装
date: 2016-03-29 17:23:24
tags:
- MySQL
- MariaDB
---

**MariaDB** 数据库管理系统是 **MySQL** 的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：Oracle公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。
—— [维基百科](https://zh.wikipedia.org/wiki/MariaDB)

<!--more-->

### 下载
![mariadb](https://mariadb.org/wp-content/uploads/2015/05/MariaDB-Foundation-horizontal-x52.png)
[MariaDB官网](https://downloads.mariadb.org/)
[mariadb-10.1.13.tar.gz](http://mirrors.opencas.cn/mariadb//mariadb-10.1.13/source/mariadb-10.1.13.tar.gz)

### 编译环境
从 `MySQL 5.5`之后，源码编译安装都需要用到`cmake`，`cmake`特性是独立于源码编译，编译工作可以在另外一个目录中而非源码目录中进行。

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y \
bison \
cmake \
libncurses5-dev \
automake \
autoconf \
libtool \
zlib1g-dev \
openssl \
unzip \
libxml2-dev \
libboost-dev \
libjudy-dev 
```

### 开始编译
下载好`mariadb-10.1.13.tar.gz`后解压，开始编译。

```bash
$ cd mariadb-10.1.13
$ cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql  \
-DMYSQL_DATADIR=/data/mysql  \
-DSYSCONFDIR=/etc/mysql \
-DMYSQL_USER=mysql \
-DWITH_MYISAM_STORAGE_ENGINE=1 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_MEMORY_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DMYSQL_UNIX_ADDR=/var/run/mysqld/mysqld.sock \
-DMYSQL_TCP_PORT=3306 \
-DENABLED_LOCAL_INFILE=1 \
-DENABLE_DOWNLOADS=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1  \
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci \
-DWITH_DEBUG=0 \
-DMYSQL_MAINTAINER_MODE=0 \
-DWITH_SSL:STRING=bundled \
-DWITH_ZLIB:STRING=bundled \
-DWITHOUT_MROONGA_STORAGE_ENGINE=1
```

`cmake`是个漫长的等待。。大概有十几分钟。中间有次报错，编译到`mroonga`引擎默认配置的时候进程被kill了，于是加了上面配置的最后一行，关闭了mroonga引擎，之后编译顺利，也不排除是内存不足的原因 : )

```bash
$ make
$ sudo make install
support-files/mysql.server to the right place for your system
PLEASE REMEMBER TO SET A PASSWORD FOR THE MariaDB root USER !
To do so, start the server, then issue the following commands:
'/usr/local/mysql/bin/mysqladmin' -u root password 'new-password'
'/usr/local/mysql/bin/mysqladmin' -u root -h opso password 'new-password'

Alternatively you can run:
'/usr/local/mysql/bin/mysql_secure_installation'

which will also give you the option of removing the test
databases and anonymous user created by default.
You can start the MariaDB daemon with:
cd '/usr/local/mysql' ; /usr/local/mysql/bin/mysqld_safe --datadir='/data/mysql'

You can test the MariaDB daemon with mysql-test-run.pl
cd '/usr/local/mysql/mysql-test' ; perl mysql-test-run.pl

Please report any problems at http://mariadb.org/jira
```

### 添加mysql环境变量

```bash
# echo -e '\n\nexport PATH=$PATH:/usr/local/mysql/bin\n' >> /etc/profile && source /etc/profile
echo -e '\n\nexport PATH=$PATH:/usr/local/mysql/bin\n' >> ~/.bashrc && source ~/.bashrc
```

### 修改权限

```bash
sudo groupadd mysql
sudo useradd -g mysql mysql
sudo mkdir -p /data/mysql
sudo mkdir /var/run/mysqld
sudo mkdir /etc/mysql
sudo chown -R mysql:mysql /data/mysql/
sudo chown -R mysql:mysql /var/run/mysqld/
```

### 添加配置文件

```bash
sudo cp support-files/my-small.cnf /etc/mysql/my.conf
```

### 修改配置文件

```cnf
[client]
default-character-set = utf8

[mysqld]
datadir = /data/mysql
character-set-server = utf8
```

### 初始化数据库

```bash
cd /usr/local/mysql
scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/data/mysql --user=mysql
```

### 开机启动

```bash
sudo cp support-files/mysql.server /etc/init.d/mysql
sudo sysv-rc-conf mysql on
sudo service mysql start
```

### 修改root密码

```bash
mysqladmin -uroot password '123456'
#或
mysql -uroot  
mysql> SET PASSWORD = PASSWORD('123456');
```

安装完成 : )