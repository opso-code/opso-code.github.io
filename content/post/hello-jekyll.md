---
layout: post
title: Jekyll搭建静态博客
author: opso
date: 2016-03-12
tag:
    - jekyll
    - github pages
    - markdown
---

**Jekyll** 是一个使用ruby语言开发的静态网页生成工具，今天使用这项技术在Github Pages上搭建静态个人博客。

<!--more-->

前几天突然收到[新浪云SAE](http://sae.sina.com.cn)将收费的邮件通知，之前一直都是在上面挂着我的个人博客，考虑到上面的豆子也不多了，还是另辟蹊径吧。:)

之前也也有听说过Github提供Github Pages服务，支持Jekyll搭建静态博客网站，静态网站的好处是轻量级的网站构造，没有动态网站的沉重，当然对于我来说另外最重要的好处就是是支持使用`Markdown`语法写博客:)

### 开始

* 安装ruby
* 使用gem安装Jekyll
* 使用Jekyll创建你的博客站点
* 开启Jekyll服务
* 使用Jekyll写博文
* 创建我们的仓库
* 为仓库创建Github Pages
* 将本地jekyll代码部署到Github上的仓库
* 克隆仓库到本地
* 拷贝本地的jekyll目录到版本库
* 本地Jekyll站点部署到Github Pages上

`Jekyll`开发环境搭建和一些基础介绍，下面的参考说的很详细
[参考资料](http://pizida.com/technology/2016/03/03/use-jekyll-create-blog-on-github/#gemjekyll)

### 启动
搭建好Jekyll开放环境后，启动Jekyll，由于我是在虚拟机(ubuntu)中开发，启动命令变为

> jekyll serve build -B -H 0.0.0.0

下面是我在目录下新建的一个重启脚本`rebuild.sh`

```sh
#!/bin/bash
sudo pkill -f jekyll
jecyll serve build -B -H 0.0.0.0
```
新建脚本后需要执行`chmod +x rebuild.sh`，然后就可以在目录下使用`./rebuild.sh`命令来启动/重启Jekyll服务了。

剩下的就是博客的迁移了，还好不多 :)

