---
layout: post
title: Git基本操作
date: 2016-01-13
author: opso
tags:
- git
---

`git`是一个快速的分布式版本控制系统。之前也有了解过，虽然现在公司里用的是`SVN`，对照着官方文档，`git`的基础入门。

<!--more-->

### Git安装
官方下载: <http://git-scm.com/download/>

### 全局配置
配置`user`和`email`，也可以在`.gitconfig`文件里修改

```bash
$ git config --global user.name "opso"
$ git config --global user.email "945206413@qq.com"
#列出config选项
$ git config --global --list
```


```bash
#初始化仓库
$ cd project
$ git init .

#通过http(s)协议 下载代码
$ git clone https://github.com/git/git.git
#通过ssh协议
$ git clone git@github.com:git/git.git

#创建和clone后默认的分支是master，默认的repository引用名称origin

#更新代码
$ git pull

#添加文件到索引
git add readme.md a.txt ...
#也可以用git add .
git add .

#查看当前状态
$ git status
#简短显示
$ git status -s

# 提交
$ git commit -m "注释内容"
#如果只是修改了文件，没有添加新的文件，可以省略git add
$ git commit -am "注释内容"

#提交代码
$ git push origin master
```


### SSH方式连接git
首先生成sshkey

```bash
ssh-keygen -t rsa -C "xxxxx@xxxxx.com"
#默认生成在用户目录/.ssh/id_rsa.pub
cat ~/.ssh/id_rsa.pub
ssh-rsa ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDTjS036fU0t4xxxxxx
...
```
然后在 https://git.oschina.net/profile/sshkeys 页面复制添加这个key，验证下

```bash
$ ssh -T git@git.oschina.net
The authenticity of host 'git.oschina.net (101.201.241.32)' can't be established.
ECDSA key fingerprint is SHA256:FQGC9Knxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'git.oschina.net,101.201.241.32' (ECDSA) to the list of known hosts.
Welcome to Git@OSC, opso!
```

返回welcome证明成功，以后pull和push就不用输密码啦 :)

### 参考

Git官方基础文档：<http://git-scm.com/book/zh/v2/>
一张图片了解git：
![git_map](/images/git_map.png)