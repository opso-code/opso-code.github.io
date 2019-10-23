---
layout: post
title: ELK初探-安装(一)
date: 2017-08-02
author: opso
tags: 
 - elk
 - elasticsearch
 - kibana
 - logstash
header-img: ELK.png
---

ELK 其实并不是一款软件，而是一整套解决方案，是三个软件产品的首字母缩写，Elasticsearch，Logstash 和 Kibana。这三款软件都是开源软件，通常是配合使用，而且又先后归于 Elastic.co 公司名下，故被简称为 ELK 协议栈

<!--more-->

## ElasticSearch
是一个基于Lucene的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于RESTful web接口。

主要特点：

- 实时分析
- 分布式实时文件存储，并将每一个字段都编入索引
- 文档导向，所有的对象全部是文档
- 高可用性，易扩展，支持集群（Cluster）、分片和复制（Shards 和 Replicas）
- 接口友好，支持 JSON

## Logstash

Logstash 是一个具有实时渠道能力的数据收集引擎。使用 JRuby 语言编写。其作者是世界著名的运维工程师乔丹西塞 (JordanSissel)

- 几乎可以访问任何数据
- 可以和多种外部应用结合
- 支持弹性扩展

## Kibana

Kibana 是一款基于 Apache 开源协议，使用 JavaScript 语言编写，为 Elasticsearch 提供分析和可视化的 Web 平台。它可以在 Elasticsearch 的索引中查找，交互数据，并生成各种维度的表图


## 安装

### 1.安装JDK

安装依赖`java`环境，这里通过命令来装

> sudo apt install openjdk-8-jdk

### 2.安装elasticsearch

可以在[官网下载](https://www.elastic.co/downloads/elasticsearch)好对应系统的包，通过tar源码安装方式，你也可以下载deb文件使用`dpkg -i elasticsearch-xxx.deb`方式安装；目前最新为5.5版本，电脑内存要求最好1.5G以上。

启动的方式也很简单

```
./elasticsearch/bin/elasticsearch
```

这里一直在提示`low disk watermark`，表示磁盘空间不太够 : )

>[1]: max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]
>[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

修改内核参数`vm.max_map_count` 进程中内存映射区域的最大数量：

```bash
$ sudo vim /etc/sysctl.conf
vm.max_map_count = 262144
$ sudo sysctl -p
$ sudo vim /etc/security/limits.conf
elasticsearch  -  nofile  65536
```

修改JVM启动内存大小

```bash
$ vim config/jvm.options  
-Xms2g  
-Xmx2g  
修改为  
-Xms512m  
-Xmx512m
```

服务默认在`localhost:9200`启动，可使用浏览器或者`curl`命令请求：

```bash
$ curl 'http://localhost:9200/?pretty'
{
  "name" : "Mp3BR27",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "oVnmBbc6SJuHtZ9qGZbqhg",
  "version" : {
    "number" : "5.5.1",
    "build_hash" : "19c13d0",
    "build_date" : "2017-07-18T20:44:24.823Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.0"
  },
  "tagline" : "You Know, for Search"
}
```

由于是在虚拟机中安装，在主机中远程访问的时候，发现端口9200无法访问。
首先将elasticsearch和kibana配置文件中host改为0.0.0.0,然后开启防火墙端口或者直接关闭防火墙，这样就可以远程访问了。

```bash
$ vim elasticsearch/config/elasticsearch.yml
network.host: 0.0.0.0
$ vim kinaba/config/kibana.yml
server.host: "0.0.0.0"
# 查看防火墙端口情况
sudo iptables -L -n 
# 开启端口
sudo iptables -A INPUT -p tcp --dport 9200 -j ACCEPT
# 或者直接关闭防火墙
$ sudo ufw disable
```

### 3.安装Kibana

[kibana-4.3.0-linux-x64.tar.gz](https://download.elastic.co/kibana/kibana/kibana-4.3.0-linux-x64.tar.gz)

解压后修改配置文件中 `server.host`：

```yml
server.host:"localhost"
```

启动

```sh
cd kibana/bin
./kibana
```

### 4.安装Logstash
[logstash-2.1.1.tar.gz](https://download.elastic.co/logstash/logstash/logstash-2.1.1.tar.gz)


接下来有时间整理下ElasticSearch常见存入查询操作