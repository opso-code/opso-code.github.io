---
layout: post
title: 记一次MySQL维护分区过程
date: 2017-03-20
author: opso
tags:
  - mysql
  - partition
cover: "/images/mysql_logo.jpg"
---
前段时间有几台MySQL服务器内容警告，已经超过了80%，达到了40G/50G，这些数据库主要是记录用户的一些操作日志，查看表大小后锁定三个数据量比较大的表。

<!--more-->

先来看下表结构

```sql
-- 表1：
CREATE TABLE `jy_cuser_item_log` (
  `user_id` int(10) NOT NULL,
  `item_id` smallint(5) NOT NULL,
  `num` int(11) DEFAULT NULL,
  `surplus` int(11) DEFAULT NULL,
  `create_time` int(10) NOT NULL,
  KEY `user_id_time` (`user_id`,`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

-- 表2：
CREATE TABLE `jy_cuser_pm` (
  `user_id` int(11) NOT NULL,
  `type` int(11) NOT NULL,
  `num` int(11) NOT NULL,
  `surplus` int(11) NOT NULL,
  `create_time` int(11) NOT NULL,
  KEY `user_id_time` (`user_id`,`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

-- 表3：
CREATE TABLE `jy_com_break_log` (
  `Id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL DEFAULT '0',
  `com_old_data` varchar(2000) NOT NULL DEFAULT '',
  `fragment_add_num` int(11) NOT NULL DEFAULT '0',
  `create_time` int(10) NOT NULL DEFAULT '0',
  PRIMARY KEY (`Id`),
  KEY `user_id` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=5932063 DEFAULT CHARSET=utf8
```

查看了表结构后发现所有的表都有`create_time`  `int` 类型时间戳字段，最早的数据能到 `2015-11-05` ，已经超过半年以上，之前也没有相关的清除数据的策略，于是决定删除 `2016-09-01` 前的数据，自然而然有了下面的语句：

```sql
DELETE FROM `jy_com_break_log` WHERE `create_time`<unix_timestamp(20160602)
```

删除了差不多35%的数据，然而查看MySQL容量，丝毫未减，原来`DELETE FROM`并不会释放表空间。新增的数据会使用这些没有释放的空间，但也不是长久之计。

在了解了分区（`PARTITION`）的好处后，决定建立新表，按时间分区，每月一个人分区，定期删除旧的分区，这样能马上释放分区且查询更快.

这里将表后缀都加上了`_2017`，同时修改程序中的指定，去除了主键，修改了索引，添加分区的字段必须是主键或是索引字段，否则将会报错:

> A PRIMARY KEY must include all columns in the table's partitioning function

```sql
CREATE TABLE `jy_cuser_item_log_2017` (
  `user_id` int(10) NOT NULL,
  `item_id` smallint(5) NOT NULL,
  `num` int(11) DEFAULT NULL,
  `surplus` int(11) DEFAULT NULL,
  `create_time` int(10) NOT NULL,
  KEY `user_id_time` (`user_id`,`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
PARTITION BY RANGE (create_time)
(
PARTITION p201704 VALUES LESS THAN (unix_timestamp(20170501))ENGINE=InnoDB,
PARTITION p201705 VALUES LESS THAN (unix_timestamp(20170601))ENGINE=InnoDB,
PARTITION p201706 VALUES LESS THAN (unix_timestamp(20170701))ENGINE=InnoDB,
PARTITION p201707 VALUES LESS THAN (unix_timestamp(20170801))ENGINE=InnoDB
);

CREATE TABLE `jy_cuser_pm_2017` (
  `user_id` int(11) NOT NULL,
  `type` int(11) NOT NULL,
  `num` int(11) NOT NULL,
  `surplus` int(11) NOT NULL,
  `create_time` int(11) NOT NULL,
  KEY `user_id_time` (`user_id`,`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
PARTITION BY RANGE (create_time)
(
PARTITION p201704 VALUES LESS THAN (unix_timestamp(20170501))ENGINE=InnoDB,
PARTITION p201705 VALUES LESS THAN (unix_timestamp(20170601))ENGINE=InnoDB,
PARTITION p201706 VALUES LESS THAN (unix_timestamp(20170701))ENGINE=InnoDB,
PARTITION p201707 VALUES LESS THAN (unix_timestamp(20170801))ENGINE=InnoDB
);

CREATE TABLE `jy_com_break_log_2017` (
  `Id` int(11) DEFAULT '0',
  `user_id` int(11) NOT NULL DEFAULT '0',
  `com_old_data` varchar(2000) NOT NULL DEFAULT '',
  `fragment_add_num` int(11) NOT NULL DEFAULT '0',
  `create_time` int(10) NOT NULL DEFAULT '0',
  KEY `user_id_time` (`user_id`,`create_time`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
PARTITION BY RANGE (create_time)
(
PARTITION p201704 VALUES LESS THAN (unix_timestamp(20170501))ENGINE=InnoDB,
PARTITION p201705 VALUES LESS THAN (unix_timestamp(20170601))ENGINE=InnoDB,
PARTITION p201706 VALUES LESS THAN (unix_timestamp(20170701))ENGINE=InnoDB,
PARTITION p201707 VALUES LESS THAN (unix_timestamp(20170801))ENGINE=InnoDB
);
```

建好表后，插入几条数据，查看分区是否成功：

```sql
SELECT
  partition_name part, 
  partition_expression expr, 
  partition_description descr, 
  table_rows 
FROM
  INFORMATION_SCHEMA.partitions 
WHERE
  TABLE_SCHEMA = schema() 
  AND TABLE_NAME='jy_com_break_log_2017';
```

```
+---------+-------------+------------+------------+
| part    | expr        | descr      | table_rows |
+---------+-------------+------------+------------+
| p201704 | create_time | 1493568000 |          3 |
| p201705 | create_time | 1496246400 |          2 |
| p201706 | create_time | 1498838400 |          0 |
| p201707 | create_time | 1501516800 |          0 |
+---------+-------------+------------+------------+
```

分区成功！之后定期删除旧的分区，就能释放表空间了！

```sql
-- 添加分区
ALTER TABLE jy_cuser_item_log_2017 ADD PARTITION (PARTITION p201708 VALUES LESS THAN (unix_timestamp(20170901)));

-- 删除分区
ALTER TABLE jy_cuser_item_log_2017 DROP PARTITION p201704;
```