---
layout: post
title: Hexo使用和nodejs环境搭建
date: 2016-03-16 21:05:10
tags: 
- hexo
- nodejs
- npm
---

**Hexo** -- A fast, simple & powerful blog framework 一个简单、强大、且快速的静态博客生成工具。

<!--more-->

相比于**Jekyll**(基于`ruby`环境)，**Hexo** 要简单方便些，首先原因
* Hexo是基于**nodejs**生成静态页面，js执行效率要比Jekyll高，占内存小，特别是文件多的时候
* Jekyll好久没更新了，中文主题资源较少，找了半天才在知乎上找到一个比较好看的；
* 其次Hexo的作者是个中国(台湾)人！ 开源的中文主题很多，中文文档也很详细。
<!--more-->
于是决定在Hexo上试试，当然首先安装nodejs！由于是在windows下，下载[node-v5.9.1-x64.msi](https://nodejs.org/dist/v5.9.1/node-v5.9.1-x64.msi)，一路next

1. [nodejs官网](https://nodejs.org/en/)

官网有`LST 4.4.1`长期支持版和`Stable 5.9.1`最新稳定版，可见nodejs是个版本狂魔。。个人建议安装最新稳定版，`npm`升级到了`3.4`，没有了windows下文件夹层数过多而无法移动的问题。

### npm常用命令

* npm config ls 列出npm配置/npm config ls -l 列出npm详细配置 也可以查看~/.npmrc
* npm config set prefix "D:\nodejs\node_global" 设置安装模块路遥
* npm config set cache "D:\nodejs\node_cache" 设置npm安装缓存目录，npm安装模块的缓存目录，离线状态可以从这里读取方便安装
* npm ls [-g] 查看安装的模块及依赖
* npm install [-g] name （全局）安装模块
* npm install name --save 当前目录下安装，并将信息添加到`package.json`文件中
* npm uninstall [-g] name （全局）卸载模块
* npm cache clean 清理缓存

2. [Hexo.io官网](https://hexo.io/zh-cn)

### 安装Hexo

```bash
$ npm install -g hexo-cli 
$ mkdir blog && cd blog
$ hexo init
$ npm install
```

这里我遇到了一个问题，由于我实在Windows下安装，`Git bash`环境下(`MINGW`)执行hexo会找不到命令，检查了几遍环境变量，添加了`NODE_PATH`，修改npm全局目录，都不行，最后才发现可能是MINGW里的环境配置没有添加。

```bash
echo $PATH
```

输出系统变量，发现的确没有Hexo模块的目录于是

``` bash
$ vim ~/.bashrc
export PATH="$PATH:/d/nodejs/node_global"
$ source /etc/profile
```

这时候再使用hexo命令终于可以了

```bash
$ hexo server
```

这时候又出现问题了，`server`参数无效。。原因是hexo 3.x以后server服务独立开了，解决办法：

```bash
$ npm install hexo-server --save
```

运行后访问`http://localhost:4000`，成功，默认主题太简单，后面在来配置和更换主题

### 配置文件_config.yml

文档里已经说的很详细了，这里就不在多说了，大多配置都很明显，这里挑几个值得注意的

```yml
# Site
title: opso's blog
subtitle: "洛阳亲友如相问，人丑就该多读书。"
description: "opso's 技术博客"
author: opso
language: 
- zh-cn
- en
timezone: Asia/Shanghai
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:
# Category & Tag
default_category: uncategorized
category_map:
tag_map:
# Archives 默认值为2，这里都修改为1，相应页面就只会列出标题，而非全文
archive: 1
category: 1
tag: 1
# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss
# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page
# theme
theme: indigo
# Deployment
deploy:
  type: git
  repository: git@github.com:opso-code/opso-code.github.io.git
  branch: master
```

### 主题和hexo部署

Hexo主题更换很方便，只要将主题文件夹下载好放到`theme`文件夹下，修改主题的`_config.yml`,然后修改根目录下的`_config.yml`里`theme`主题名。

```bash
$ npm install hexo-deployer-git --save
$ hexo deploy
```

将生成的静态页面文件自动提交到GIT上，github地址可以在`_config.yml`里配置，非常方便，
配置好`Git pages`上传之后过会就可以访问了 :)
