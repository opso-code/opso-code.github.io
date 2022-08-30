---
layout: post
title: "Windows10 开启 Bash on Ubuntu"
date: 2016-09-20
author: opso
tags:
- bash on windows
---

微软已经在北京时间8月3日开始推送Win10一周年更新正式版，版本号为1607，增加了许多新的特性，包括Edge浏览器扩展和Linux子系统（ubuntu）。今天心血来潮，体验下一周年新特性 —— Bash on Ubuntu on Windows。

这是一项能让Ubuntu作为Windows子系统运行的黑科技，这个东西表面上看就像个模拟器，类似cygwin，也可能有人会想到虚拟机(之前用过vagrant，也不错)，但是它实际上要更直接，它是在用 Windows 内核实现了对 Linux 系统调用的兼容支持，性能接近原生，看来微软对于Docker能在Windows上运行努力地铺路啊。。

<!--more-->

## 安装

下面我们来说说如何在win10 一周年更新上愉快的玩耍ubuntu

首先 你需要以下条件:

> 1. 确认系统已经更新到Windows 10 1607版本
> 2. 64位处理器 & 64位操作系统

我们可以打开设置/系统/关于，如图：

![查看windows版本](/images/md_win_about.png)

我的 win10 已经安装完一周年更新，首先启用（适用于Linux的子系统），在控制面板/启用或关闭Windows功能,找到 `适用于Linux的子系统` ,勾选后要求重启。

![启用或关闭Windows功能](/images/md_open_ubuntu_function.png)

重启后打开 Windows PowerShell，输入 `bash` ，发现需要开启开发者模式。

所有设置/安全和更新/针对开发人员/勾选开发人员模式

其中有个警告确认框，意思就是开发人员选项将开启新的特性，可能不安全，点确认就行，
输入 `bash` ，要求下载相关组件，输入 `y` ,回车确认下载。

下载完后就是按照提示输入用户名/密码，之后就可以愉快的玩耍啦~

![安装](/images/md_bash_on_ubuntu.png)

## 卸载

如果需要卸载Bash on Ubuntu on Windows的话，以管理员身份运行命令提示符，或者以管理员身份运行Windows Powershell
执行 `lxrun /uninstall /full` 回车，这个会删除所有Bash on Ubuntu下的文件、账户和设置。

## 其他

推荐一款Windows下类Unix终端，超越 CMD/Windows Powershell 的神奇 —— [Cmder](http://cmder.net/)

官网有两个版本，这里我只需要下载 mini 绿色版

在设置里，Startup 选项 Command line 勾选并添加 `%windir%\system32\bash.exe -cur_console:p1`,就可以在 Cmder 里默认登陆 ubuntu 子系统了

![cmder](/images/md_cmder.png)

接着把 Cmder 添加到上下文环境右键菜单中，方便在任何文件夹右键打开 Cmber，首先把 Cmder 添加到系统变量。

用管理员权限代开命令提示符（CMD），输入 `Cmder.exe /REGISTER ALL` ，运行后就可以在文件夹右键选项看到了。
