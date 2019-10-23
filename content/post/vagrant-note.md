---
layout: post
title: 用Vagrant搭建跨平台ubuntu开发环境
date: 2015-12-20
author: opso
tags: 
    - vagrant
    - ubuntu
---

Vagrant是一款用来构建虚拟开发环境的工具，非常适合php/python/ruby/java这类语言开发，代码在我机子上运行没有问题`这种说辞将成为历史。

<!--more-->

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 223 60" class="logo" height="100">
  <path class="text" fill="#000000" d="M99.88 14.1h6.29l-9.56 32h-8.93l-9.56-32h6.29l7.73 26.66 7.74-26.66zm23.78 32h-4.8l-.43-1.59c-2.083 1.355-4.515 2.074-7 2.07-4.28 0-6.1-2.93-6.1-7 0-4.76 2.07-6.58 6.82-6.58h5.62v-2.42c0-2.59-.72-3.51-4.47-3.51-2.182.023-4.356.264-6.49.72l-.72-4.47c2.606-.723 5.296-1.096 8-1.11 7.35 0 9.51 2.59 9.51 8.46l.06 15.43zm-5.86-8.84h-4.32c-1.92 0-2.45.53-2.45 2.31s.53 2.35 2.35 2.35c1.55-.022 3.07-.435 4.42-1.2v-3.46zm17.05 2.54c-.74.36-1.26 1.057-1.39 1.87 0 .62.38.91 1.3 1 2.59.29 4 .43 6.77.72 3.8.43 5 2.31 5 5.67 0 5-1.83 7-10.57 7-3.04.003-6.067-.4-9-1.2l.72-4.37c2.576.654 5.222.99 7.88 1 4.66 0 5.57-.34 5.57-1.87s-.43-1.68-2.21-1.87c-2.69-.29-3.8-.43-6.77-.77-3.31-.38-4.61-1.49-4.61-4.47.102-1.657 1.02-3.156 2.45-4-2.16-1.3-3.17-3.46-3.17-6.29V30c.1-4.85 2.64-7.78 9.42-7.78 1.348-.01 2.692.15 4 .48h7.21v2.93c-.82.24-1.78.48-2.59.72.558 1.135.84 2.386.82 3.65v2.21c0 4.76-2.88 7.64-9.42 7.64-.47.01-.94-.006-1.41-.05zm1.34-12.88c-2.88 0-3.89 1.06-3.89 3.27V32c0 2.31 1.15 3.17 3.89 3.17s3.94-.91 3.94-3.17v-1.81c.01-2.19-1-3.26-3.93-3.26l-.01-.01zm25.9.68c-2.148.973-4.217 2.11-6.19 3.4v15.1H150V22.7h5l.38 2.59c1.906-1.29 3.974-2.32 6.15-3.07l.56 5.38zm19.6 18.5h-4.8l-.43-1.59c-2.083 1.355-4.515 2.074-7 2.07-4.28 0-6.1-2.93-6.1-7 0-4.76 2.07-6.58 6.82-6.58h5.62v-2.42c0-2.59-.72-3.51-4.47-3.51-2.182.023-4.356.264-6.49.72l-.72-4.47c2.606-.723 5.296-1.096 8-1.11 7.35 0 9.51 2.59 9.51 8.46l.06 15.43zm-5.86-8.84h-4.32c-1.92 0-2.45.53-2.45 2.31s.53 2.35 2.35 2.35c1.55-.022 3.07-.435 4.42-1.2v-3.46zm23.3 8.84V29.76c0-1.25-.53-1.87-1.87-1.87-2.147.252-4.22.932-6.1 2V46.1h-5.86V22.7h4.47l.58 2c2.92-1.46 6.11-2.295 9.37-2.45 3.89 0 5.28 2.74 5.28 6.92v17l-5.87-.07zm23.11-.44c-1.653.578-3.39.886-5.14.91-4.28 0-6.44-2-6.44-6.2v-13h-3.51V22.7h3.51v-5.81l5.86-.82v6.63h6l-.38 4.66h-5.62v12.25c-.076.576.124 1.154.54 1.56.414.408.995.597 1.57.51.994-.03 1.98-.19 2.93-.48l.68 4.46z"></path>
  <path class="front" fill="#1563FF" d="M58.03 10.12V4.63L44.84 12.3v4.64L34.29 39.7l-5.28 3.64v16.67l11.31-6.52 17.71-43.37zM29.01 31.47L21.1 13V7.78l-.05-.02-7.86 4.54v4.64L23.74 40.7l5.27-2.61v-6.62z"></path>
  <path class="shadow" fill="#104EB2" d="M50.12.01L36.94 7.73h-.01V13l-7.92 18.47v6.17l-5.27 3.06-10.55-23.76v-4.65l7.92-4.55L7.91.01 0 4.63v5.66l17.81 43.25 11.2 6.47V43.76l5.28-3.06-.07-.04 10.62-23.72v-4.65l13.19-7.66"></path>
</svg>

**Vagrant** 是一个基于`Ruby`的工具，用于创建和部署虚拟化开发环境。它 使用`Oracle`的开源`VirtualBox`虚拟化系统，使用`Chef`创建自动化虚拟环境。

随着容器技术越来越火，很多其他技术都是基于linux环境实现，想在自己的电脑上搭建一个linux环境，在了解了`Cygwin`和`MSYS2`后，感觉都不是我的菜。。看来看去感觉还是虚拟机比较实在，平时自己开发研究些东西更方便，一边下载系统镜像文件一边搜瞎逛，发现了很厉害的东西。。

首先下载好`VirtualBox 5.0` 和 `Vagrant 1.8`，安装过程略 :)

1. [Vagrant官网下载](https://www.vagrantup.com/downloads.html)
2. [VisualBox官网下载](https://www.virtualbox.org/wiki/Downloads)

### 常用命令

```bash
#添加box镜像到 Vagrant
vagrant box add xxxx.box
#列出所有已添加的镜像
vagrant box list
#初始化Vagrantfile
vagrant init
#删除当前路径下的虚拟机
vagrant destroy
#ssh登录虚拟机
vagrant ssh
#开机
vagrant up
#关机
vagrant halt
#重启
vagrant reload
#打包到packages.box文件
vagrant package 
```

Vagrant初始化

```bash
mkdir workspace
cd workspace
#生成Vagrantfile,开始下载 ubuntu 14.04 "镜像"
vagrant init ubuntu/trusty64
```
box 镜像文件可以在 [Vagrantbox.es](http://www.vagrantbox.es/) 下载，鉴于国内网络环境，建议提前下载好 :)

### Vagrantfile配置

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"
  config.vm.hostname = "ubuntu" #命令行开头用户名

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
   config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
   config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
     vb.memory = "1024"
     vb.name = "ubuntu" #虚拟机名（虚拟机文件夹名）
     vb.cpus = 2
   end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
```
配置尽量默认，开机

```bash
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Clearing any previously set forwarded ports...
==> default: Clearing any previously set network interfaces...
==> default: Preparing network interfaces based on configuration...
    default: Adapter 1: nat
    default: Adapter 2: hostonly
==> default: Forwarding ports...
    default: 22 (guest) => 2222 (host) (adapter 1)
==> default: Running 'pre-boot' VM customizations...
==> default: Booting VM...
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
==> default: Checking for guest additions in VM...
    default: The guest additions on this VM do not match the installed version of
    default: VirtualBox! In most cases this is fine, but in rare cases it can
    default: prevent things such as shared folders from working properly. If you see
    default: shared folder errors, please make sure the guest additions within the
    default: virtual machine match the version of VirtualBox you have installed on
    default: your host and reload your VM.
    default:
    default: Guest Additions Version: 4.3.34
    default: VirtualBox Version: 5.0
==> default: Setting hostname...
==> default: Configuring and enabling network interfaces...
==> default: Mounting shared folders...
    default: /vagrant => D:/workspace
==> default: Machine already provisioned. Run `vagrant provision` or use the `--provision`
==> default: flag to force provisioning. Provisioners marked to run always will still run.
```
这里提示虚拟机的 `guest additions`插件和`VirtualBox` 上的不一致(由于我是升级的visualbox)一般不影响使用，有强迫症的我，决定升级这个插件。

```bash
#在Vagrantfile目录下
vagrant plugin install vagrant-vbguest
```
这里也会编译虚拟机内核。

```bash
vagrant up
sudo apt-get clean
sudo apt-get autoclean
#删除旧内核的不用的包
sudo apt-get autoremove
```

在windows下不能通过 `vagrant ssh` 来登录虚拟机，推荐使用`Git for Windows`的命令行工具（MINGW），或者下载`xshell 5`通过ssh连接虚拟机，默认用户名是vagrant，密码vagrant

### 搭建LNMP环境

```bash
sudo apt-get install mysql-server mysql-client nginx php5-fpm memcached
#添加php扩展
sudo apt-get install php5-curl php5-gd php5-mysql php5-memcache php5-tidy
#安装git
sudo apt-get install git
#安装nginx
sudo apt-get install nginx
#安装composer
sudo curl -sS https://getcomposer.org/installer | sudo php -d detect_unicode=Off
sudo chmod a+x composer.phar
sudo mv composer.phar /usr/local/bin/composer
#升级包
sudo composer self-update
#创建Laravel项目
sudo composer create-project laravel/laravel laravelTest --prefer-dist
```

**Vagrant** 的一个非常重要的功能就是在你的同事之间分享你的box从而使大家的开发环境保持同步，打包［package］正是实现这一功能的关键所在

### 打包分发

`vagrant package`的命令很简单，不过有些地方官方文档并没有说清楚，我在这里补充一下

```bash
vagrant package -h
Usage: vagrant package [options] [name]
Options:
        --base NAME     virtualbox程序里面的虚拟机的名称，不是box的名字也不是Vagrantfile里面的虚拟机名称.默认是打包当前目录下面的虚拟机。
        --output NAME   要打包成的box名称，不会自动添加.box后缀，要手动加.默认值package.box
        --include FILE...   打包时包含的文件名，你可以把.box文件理解为一个压缩包
        --vagrantfile FILE  打包时包含的Vagrantfile文件，原理和上面类似
    -h, --help              Print this help
```

### 【2016-1-20 更新】 Ubuntu下LNMP环境搭建

[PHP7安装笔记](../php7-install)
