---
layout: post
title: ELK实践-nginx日志分析(二)
date: 2018-08-02
author: opso
tags: 
 - elk
 - elasticsearch
 - kibana
 - logstash
header-img: ELK.png
---

之前有写过ELK的介绍和安装 [ELK初探-安装(一)](../elasticsearch_note/)，在熟悉了Docker操作后，快速搭建了一个ELK日志分析环境，作用是由 `logstash` 读取 `nginx` 日志文件到 `elasticsearch` 中，通过 `kibana` 展示查询，下面来用两种格式的日志来说明:

<!--more-->

Docker使用的是 `sebp/elk` 镜像， `docker-compose.yaml` 如下：

```yaml
version: '3'
networks:
  elk-net:
    driver: bridge
services:
  elk:
    image: sebp/elk
    container_name: elk
    volumes:
      - /data/volumes/log/nginx:/var/log/nginx
      - /data/volumes/log/transfer.log:/var/log/transfer.log
      - /data/volumes/elasticsearch:/var/lib/elasticsearch
      - ./logstash/conf.d:/etc/logstash/conf.d
    environment:
      - "TZ=Asia/Shanghai"
      - "ES_HEAP_SIZE=512m"
      - "LS_HEAP_SIZE=256m"
    ports:
      - "5601:5601"
      - "9200:9200"
      - "5044:5044"
    networks:
      - elk-net
```


## 解析nginx-access访问日志

首先我们要确定nginx日志的位置和日志格式，可以在nginx主配置文件中确认，我这里日志位置为`/var/log/nginx/access.log`

```nginx
# nginx.conf
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    include /etc/nginx/conf.d/*.conf;
}
```

`log_format` 就是我们要找的日志格式格式，所以日志开起来是这个样子的，logstash有预先定义好的 `${NGINXACCESS}` pattern解析式，可惜跟我这个格式不一样，所以得自己来解析了。

```text
192.168.33.1 - - [08/Aug/2018:11:53:38 +0800] "GET http://docker.test/test.php HTTP/1.1" 200 293 "http://docker.test/" "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36" "-"
```

Docker启动elk就在这忽略了，下面来针对这个日志格式来写 `logstash` 的配置

```conf
# nginx-access.conf
input {
  file {
    path => "/var/log/nginx/access.log"
    type => "nginx-access"
    start_position => beginning
  }
}
filter {
  if [type] == "nginx-access" {
    grok {
      match => {
        "message" => "%{IPORHOST:http_host} - %{USER:remote_user} \[%{HTTPDATE:[@metadata][timestamp]}\] \"(?:%{WORD:method} %{NOTSPACE:request}(?: HTTP/%{NUMBER:http_version})?|%{DATA:raw_http_request})\" %{NUMBER:status} (?:%{NUMBER:body_bytes_sent}|-) \"%{NOTSPACE:referrer}\" \"%{DATA:agent}\" \"%{NOTSPACE:xforwardedfor}\""
      }
      remove_field => ["message", "@version"]
    }
    date {
      match => ["[@metadata][timestamp]", "dd/MMM/yyyy:HH:mm:ss Z"]
    }
    mutate {
      convert => {
        "body_bytes_sent" => "integer"
        "http_version" => "float"
        "status" => "integer"
      }
    }
    useragent {
      target => "useragent"
      source => "agent"
    }
  }
}
output {
  if [type] == "nginx-access" {
    elasticsearch {
      hosts => ["localhost:9200"]
      index => "%{type}-%{+YYYY.MM.dd}"
    }
  #stdout { codec => rubydebug }
  }
}
```

`logstash` 配置文件目录均放到了 `/etc/logstash/conf.d`  ，配置文件主要分为三大部分。

### input 日志输入

`input` 表示日志来源。由于使用的是nginx日志文件作为日志源，使用 `file`模块：

- `path` 日志位置，注意日志位置必须绝对路径。
- `type` 表示当前`input`的标识，后面解析和输出的时候会用到。
- `start_position` 标识开始读的位置，`beging` 标识从文件开头开始解析

### filter 日志解析

`grok` 插件配置，是对日志内容进行解析、筛选、修改等操作的位置。

- `match` 使用`grok`的匹配规则来解析日志内容，也可以使用自定义正则表达式。
- `date` 对解析结果中的`@timestamp` 默认字段进行修改。`@timestamp` 是logstash解析日志默认的字段，kibana中的查询默认都是以这个字段来作为时间参数，如果我们不想让当前时间作为日志发生的时间来查询，这里就需要替换 `@timestamp` 为`[@metadata][timestamp]` 中的时间，且 `[@metadata][timestamp]` 的格式为 `dd/MMM/yyyy:HH:mm:ss Z`
- `mutate` 对解析的数据字段进行修改，这里使用了 `convert` 进行了类型转换
- `useragent` logstash自带的一个解析插件，自动对日志中的nginx的浏览器agent进行解析

### output 解析后输出

解析结果输出类型，可以是文件、redis、tcp、elasticsearch等等

> 注意：这里对 `input` 模块中定义的 `type` 属性进行了判断，多解析配置的情况下防止误输出

到这里就配置好了，其他配置选项也可以去[Logstash官方文档](https://www.elastic.co/guide/en/logstash/current/index.html)查看，这里只是用到了一部分。

elk全都运行起来后，在 `kibana` 中建立 `nginx-access-*` 的索引就可以查询了：

![kibana-nginx](/images/logstash-nginx-log.png)

## 解析自定义格式文件

首先开看下日志格式：

```txt
[2018-07-26 14:22:49]  >> to:104578619,type:email,uid:102108852,exp:6004,level:8,base_coalaa:0,bank_coalaa:0,base_money:65004,bank_money:0,prop2:145,prop30:0,prop5:0,bronze:0,silver:0,gold:0,diamond:0,money:65004,coalaa:0
```

```conf
input {
  file {
    path => "/var/log/nginx/transfer.log"
    type => "transfer-log"
    start_position => beginning
  }
}
filter {
  if [type] == "transfer-log" {
    grok {
      match => {
        "message" => "\[%{TIMESTAMP_ISO8601:[@metadata][timestamp]}\]%{SPACE}>>%{SPACE}%{GREEDYDATA:[@metadata][keyvalue]}"
      }
      remove_field => ["message", "@version", "tags"]
    }
    kv {
      source => "[@metadata][keyvalue]"
      field_split => ","
      value_split => ":"
    }
    # type 字段被覆盖了
    if [type] != "tranfer-log" {
      mutate {
        rename => { "type" => "t_type" }
        add_field => { "type" => "transfer-log" }
      }
    }
    date {
      match => ["[@metadata][timestamp]", "yyyy-MM-dd HH:mm:ss"]
    }
  }
}
output {
  if [type] == "transfer-log" {
    elasticsearch {
      hosts => ["localhost:9200"]
      index => "%{type}-%{+YYYY.MM.dd}"
    }
    # stdout { codec => rubydebug }
    file {
      path => "/var/log/nginx/result.log"
    }
  }
}
```

这里难点主要是对 `a:1,b:2,c:xx` 这样格式的字段进行切割

```conf
kv {
  source => "[@metadata][keyvalue]"
  field_split => ","
  value_split => ":"
}
```

> 注意：这里有个问题，解析后 `input` 中的 `type` 的值会被日志中的的 `type` 覆盖掉，所以这里需要把 `type` 重名为`t_type` ，再把 `type` 加回去。。



好了，到这里基本的文件日志解析就可以了，复杂的日志结构则需要引入`elasticsearch` 的模板 `template`，否则会出现 `kibana` 没有解析的内容哦，这就是另一部分内容了。