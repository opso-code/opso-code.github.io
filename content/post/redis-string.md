---
layout: post
title: Redis基本操作
date: 2015-10-15
author: opso
tags:
- redis
---

Redis中的类型，String字符串、 Hash哈希、 List列表、 Set集合、 Sort Set有序集合

<!--more-->

### String类型操作

* set key value 如果key不存在，新增；否则，更新
* setnx set if not exists 如果key不存在则新增
* setex key seconds value 设置key的过期时间和值
* mset key value [key value] 同时设置多个键值对
* msetnx 所有key都不存在时才执行set操作
* get key 获取key对应的值
* mget key [key] 批量获取
* getrange key start end 获取key对应的value并截取(0是第一个字符，-1是最后一个字符)
* getset 设置key的值，并返回key的旧值，如果可以不存在则set
* append key存在则在旧值的后面追加value，key不存在则set；返回长度
* setrange key offset value 替换key的部分子串
* incr/decr 递增/递减，返回值；如果key不存在，则视为初始值为0。
* incrby/decrby key increment 如果increment是负数则减少。
* del 删除
* strlen 获取value长度

### Hash类型操作

* hset key field value 如果key不存在，新增；否则，更新(返回int 1成功)
* hmset key field value [field value] 批量设置
* hsetnx set if not exists 如果key不存在则执行新增
* hget
* hgetall 获取key的所有字段和值
