---
layout: post
title: Linux初始化常见方式
date: 2018-03-05
author: opso
tags: [linux, init]
cover: "/images/linux_620x320.jpg"
---

近年来，Linux 系统的 init 进程经历了两次重大的演进，传统的 `sysvinit` 已经逐渐淡出历史舞台，新的 `UpStart` 和 `systemd` 各有特点，越来越多的 Linux 发行版采纳了 `systemd` (**Ubuntu15.04+/CentOS7+**)。

<!--more-->

### Sysvint
`Sysvinit` 有比较历史悠久，其主要就是一个Shell脚本，并且是放置在 `/etc/init.d` 文件夹下。然后通过`update-rc.d` 命令进行运行级别的操作来达到服务的启动。

### Upstart
`UpStart` 解决了之前提到的 `sysvinit` 的缺点。采用事件驱动模型，`UpStart` 可以：

- 更快地启动系统
- 当新硬件被发现时动态启动服务
- 硬件被拔除时动态停止服务

这些特点使得 `UpStart` 可以很好地应用在桌面或者便携式系统中，处理这些系统中的动态硬件插拔特性。

### Systemd

1. 更快的启动速度

`Systemd` 提供了比 `UpStart` 更激进的并行启动能力，采用了 `socket` / `D-Bus activation` 等技术启动服务。

为了减少系统启动时间，`systemd` 的目标是：
- 尽可能启动更少的进程
- 尽可能将更多进程并行启动

同样地，`UpStart` 也试图实现这两个目标。`UpStart` 采用事件驱动机制，服务可以暂不启动，当需要的时候才通过事件触发其启动，这符合第一个设计目标；此外，不相干的服务可以并行启动，这也实现了第二个目标。

- `Systemd` 提供了和 `Sysvinit` 以及 `LSB initscripts` 兼容的特性 
- `Systemd` 自带日志服务 `journald`，该日志服务的设计初衷是克服现有的 `syslog` 服务的缺点

### 资料

- [维基百科init系统](https://zh.wikipedia.org/wiki/Init)
- [浅析 Linux 初始化 init 系统01](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init1)
- [浅析 Linux 初始化 init 系统02](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init2)
- [浅析 Linux 初始化 init 系统03](http://www.ibm.com/developerworks/cn/linux/1407_liuming_init3)