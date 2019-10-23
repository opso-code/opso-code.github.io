---
layout: post
title: Linux swap虚拟内存设置
date: 2016-05-25
author: opso
tags:
    - swap
    - linux
cover: "/images/linux_1200x500.jpg"
---

在编译程序经常会出现

> virtual memory exhausted: Cannot allocate memory

的异常，表示虚拟内存不足，通常是没有这是swap缓存或者缓存过小，在编译安装的时候经常会遇到这样的报错，需要设置swap虚拟内存。

<!--more-->

我是在虚拟机中安装的`ubuntu 14.04` ，可以用`free -m`命令查看是否有设置swap缓存。

```bash
$ free -m
             total       used       free     shared    buffers     cached
Mem:           993        187        806          0          6         45
-/+ buffers/cache:        135        858
Swap:            0          0          0
$ sudo mkdir /opt/images
$ sudo dd if=/dev/zero of=/opt/images/swap bs=1024 count=2048000
2048000+0 records in
2048000+0 records out
2097152000 bytes (2.1 GB) copied, 4.69898 s, 446 MB/s
$ mkswap /opt/images/swap
/opt/images/swap: Permission denied
$ sudo mkswap /opt/images/swap
Setting up swapspace version 1, size = 2047996 KiB
no label, UUID=a8a831fb-a691-485f-aec0-cf94163a35e4
$ sudo swapon /opt/images/swap
$ free -m
             total       used       free     shared    buffers     cached
Mem:           993        928         64          0          0        771
-/+ buffers/cache:        156        836
Swap:         1999          0       1999
```

之后编译正常，如果嫌swap占用空间的话，编译完成后也可以关掉或者删除。

```bash
$ sudo swapoff swap  
$ sudo rm -f /opt/images/swap 
```