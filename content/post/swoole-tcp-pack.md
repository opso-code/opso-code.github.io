---
title: "通过swoole理解TCP粘包"
author: "opso"
cover: "/images/swoole_logo.png"
tags: ["swoole", "tcp"]
date: 2019-11-25T15:14:09+08:00
draft: false
---

## 什么是粘包

**TCP**是字节流协议，数据传输像流水一样。数据发送者会存在一个发送缓冲区，每次可能会将多个数据包一次性发送出去，也可能是一个数据包拆分多次发出去；如果协议没有约定好分隔符或者不明确数据包大小边界，接收者就无法获取并正确解析数据包了，这就是常说的 **粘包**，下面结合`swoole_server`和自定义协议来详细说明。

<!--more-->

## 一、 常见的自定义协议

### 1. 固定包大小

发送者和接收者约定数据包长度，适合需求较简单，消息类型固定的情况，灵活性很差。

### 2. EOF结束符

发送者和接收者约定数据包已一个特殊的结束符(**EOF**)做结尾。适合协议相对简单的需求，常见的比如 `redis`、`memcache`、`ftp`、`stmp`等都是用`\r\n`换行符作为结束符。

### 3. 固定包头+包体协议

发送者和接收者约定数据开始于一个固定长度的消息头（header），消息头里包含包体（body）的长度数据和一些其他的自定义校验数据，这样接收者就能从数据流中，根据包体的长度来截取剩下的数据包。这样的自定义协议更加灵活，且便于接收者分割数据。

现在我们来定义一个业务自己的协议，使用固定包头+包体的格式。

- 固定包头长度8
- 前6个字节表示协议头，固定为字符 `SWOOLE`
- 后2个字节为`int16`，无符号短整型(16位，大端字节序) ，表示后面的包体长度。

如图：

<img src="/images/swoole_pack.png" style="margin:0;box-shadow:none;" alt="swoole_pack"/>

这里涉及到大小端字节序的问题，推荐看看知乎专栏 [“字节序”是个什么鬼？](https://zhuanlan.zhihu.com/p/21388517)

## 二、Swoole\Server

使用`swoole_server` 我们能很轻易的实现一个TCP服务，使用者无需关注底层实现细节，就能达到使用 `TCP`/`UDP`/`UnixSocket` 搭建异步服务器的要求；在php中，一般使用 [`pack/unpack`](https://www.php.net/manual/zh/function.pack.php) 方法封包解包二进制数据，再通过swoole发送出去。

在`swoole_server->set()` 方法中提供了一些自定义协议的 [属性设置]( https://wiki.swoole.com/wiki/page/287.html)，这些设置在底层已经帮你封装好了，只要简单的设置，swoole基本不用考虑粘包问题了，真好 :tada: 

根据我们上面自定义的协议：

- 索引 `0` 到 `5` 用于协议头校验，
- 索引 `6` 和 `7` 两位表示包体长度，
- 设置 `package_length_type` 为 `n` ，表示无符号短整型大端序，
- 设置 `package_length_offset` 为 `6`，表示从索引 `6` 开始是表示包体大小的开始索引
- 设置 `package_body_offset` 为 `8`，表示以索引 `8` 为包体大小的结束索引，配合上面的`package_length_offset`，就能确定包体长度的数据在索引 `6` 到 `8` 之间，占 `2` 个字节
- 截取`6`到`8`之间的数据，根据 `package_length_type` 里的类型，转换为短整型就是包体的长度

剩下的拆包分发，swoole都会帮你做好；这样，在 `onReceive` 回调中接收到的参数 `$data`，永远会是一个完整的协议包。

### package_length_type

> 长度值的类型，接受一个字符参数，与php的 pack 函数一致。目前Swoole支持10种类型：

- `c`：有符号、`1`字节
- `C`：无符号、`1`字节
- `s`：有符号、主机字节序、`2`字节
- `S`：无符号、主机字节序、`2`字节
- `n`：无符号、网络字节序、`2`字节 (常用)
- `N`：无符号、网络字节序、`4`字节 (常用)
- `l`：有符号、主机字节序、`4`字节(小写L)
- `L`：无符号、主机字节序、`4`字节(大写L)
- `v`：无符号、小端字节序、`2`字节
- `V`：无符号、小端字节序、`4`字节

```php
<?php
// server.php
$serv = new Swoole\Server("127.0.0.1", 9501);
$serv->set(array(
    'open_length_check' => true,
    'package_length_type' => 'n',
    'package_length_offset' => 6,
    'package_body_offset' => 8,
    'package_max_length' => 2000,
));
$serv->on('Connect', function ($serv, $fd) {
    echo "Client: Connect.\n";
});
$serv->on('Receive', function ($serv, $fd, $from_id, $data) {
	$header = substr($data, 0, 8);
	$p = unpack('a6begin/nbodyLen', $header);
	if ($p['begin'] != 'SWOOLE'){
		return;
	}
	$len = $p['bodyLen'];
	$bodyPack = unpack("a{$len}body", substr($data, 8, $len));
    $serv->send($fd, "Server: ".$bodyPack['body']."\n");
});
$serv->on('Close', function ($serv, $fd) {
    echo "Client: Close.\n";
});
$serv->start();
```

```php
<?php
//client.php
$client = new swoole_client(SWOOLE_SOCK_TCP);
if (!$client->connect('127.0.0.1', 9501)) {
	exit("connect failed. Error: {$client->errCode}\n");
}
$msg = 'Hello World!';
$client->send(sendMsg($msg)); // 正常发包
$client->send(sendMsg($msg.0).sendMsg($msg.1).sendMsg($msg.2));// 模拟粘包
echo $client->recv();
$client->close();

function sendMsg($msg) {
	$p = 'SWOOLE';
	$p .= pack('n', strlen($msg));
	$p .= pack('a' . strlen($msg), $msg);
	return $p;
}
```

试试去掉 `$serv->set()` 属性设置，看看有啥不同  :smile:

## 附录：

- php pack()方法：https://www.php.net/manual/zh/function.pack.php
- swoole_server文档：https://wiki.swoole.com/wiki/page/287.html
