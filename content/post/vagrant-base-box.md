---
layout: post
title: 制作Ubuntu16.04 vagrant base box
date: 2016-04-24
author: opso
tags:
- vagrant
- ubuntu
---

前几天，期待已久的 **Ubuntu 16.04 LTS** (Xenial 好客的非洲松鼠)长期支持版正式发布了！迫不及待的下载好了Ubuntu麒麟版(Kylin)系统镜像，和win10双系统共存，试着将自己的工作环境切换到`ubuntu kylin`下，不过最终放弃 :( 

<!--more-->

原因其实很简单，中文字体和输入法阻碍了编程的激情。。对于平常PHP开发工作来说，已够用，LibreOffice、NetBeans、svn都和windows下没太大区别或者都找到了替代，让我决定放弃将ubuntu作为我的工作系统主要是`wine qq`和`Sublime Text`没办法用中文输入法。。目前还是让他运行在虚拟机里吧。
于是决定自己制作基于`ubuntu server 16.04 lts`官方镜像的vagrant base box。

## 安装配置

1. [官网](http://www.ubuntu.com/download/server)下载好`ubuntu-16.04-server-amd64.iso`系统镜像文件
2. 配置里使用网络默认配置 网络地址转换NAT
3. 禁用声音选项
4. username和password、root password都为vagrant(重要)

安装的时候发现一个问题，全中文安装会出现安装不了的bosybox报错，这里我在第一个Language语言选择English，安装语言选择中文简体，键盘布局都选择的English(US)，保留了界面语言为简体中文，在系统中会出现中文乱码方块。不过通过SSH连接中文正常。

![language](/images/16-4-24/87595680.jpg)

## 权限配置

```bash
$ sudo su -
#增加admin用户组
$ groupadd admin
$ usermod -G admin vagrant
$ visudo
##或者sudo vim /etc/sudousers :wq!强制保存
```
修改默认配置，这样`sudo`命令不需要密码

```
Defaults env_keep="SSH_AUTH_SOCK"
%admin ALL=(ALL) NOPASSWD: ALL
%sudo ALL=(ALL:ALL) NOPASSWD: ALL
```

## SSH配置

```bash
$ apt install puppet puppetmaster openssh-server
$ mkdir -p /home/vagrant/.ssh
#配置vagrant 公共秘钥
$ wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O /home/vagrant/.ssh/authorized_keys
$ chmod 0700 /home/vagrant/.ssh
$ chmod 0600 /home/vagrant/.ssh/authorized_keys
$ chown -R vagrant /home/vagrant/.ssh
```

## 安装Guest Additions
![Guest Additions](/images/16-4-24/51945902.jpg)

要使虚拟机支持共享文件夹，端口转发等一些功能需要安装增强功能 **Guest Additions** 

```bash
$ apt install -y dkms build-essential linux-headers-$(uname -r)
# 安装完后重启
$ reboot
$ sudo mount /dev/cdrom /media/cdrom
$ cd /media/cdrom
$ sudo sh VBoxLinuxAdditions.run
```

## 清除缓存

```bash
$ sudo apt clean
$ sudo apt autoclean
$ sudo apt autoremove
#下面这个很有必要！、、
$ sudo dd if=/dev/zero of=/EMPTY bs=1M
$ sudo rm -f /EMPTY
$ sudo shutdown -h now
```

## 打包成box

进入虚拟机安装目录，我的是`C:\VMs\xenial`
```bash
vagrant package --output ubuntu16.04.box --base xenial
```

打包完后发现才570M，原版iso镜像都655M，试验下

```bash
vagrant box add ubuntu16.04 xenial.box
```

Vagrantfile

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu16.04"
  config.vm.hostname = "opso"
  config.vm.network "private_network", ip: "192.168.33.10"
   config.vm.provider "virtualbox" do |vb|
     vb.memory = "1024"
	 vb.name = "codeBox"
   end
end
```

```bash
$ vagrant up && vagrant ssh
```

一切正常，中文无乱码~

## 参考
<http://aruizca.com/steps-to-create-a-vagrant-base-box-with-ubuntu-14-04-desktop-gui-and-virtualbox/>