---
layout: post
title: Travis CI自动化部署hugo静态博客
date: 2019-10-22
author: opso
tags: 
- hugo
- travis-CI
---

## Travis CI是什么

<img src="/images/travis_ci_logo.png" style="margin:0;box-shadow:none;" alt="tracis_ci_logo"/>

> Travis CI是在软件开发领域中的一个在线的，分布式的持续集成服务，用来构建及测试在GitHub托管的代码。这个软件的代码同时也是开源的，可以在GitHub上下载到，尽管开发者当前并不推荐在闭源项目中单独使用它。——维基百科

<!--more-->

在一些开源的github项目说明中，我们经常会看到 <embed src="/images/passing.svg" /> 这样的标签，表示该项目使用了**Travis CI**持续集成服务。

结合 **Travis CI**(Continuous Integration)， 它能够检测到你更新了github仓库代码，从github拉取最新代码,在自己的虚拟服务器上运行构建命令(根据`.travis.yml`)， 再将构建的结果或包，提交到github的特定分支，或是realese列表，或是其他server服务器。

简单来说，Travis CI 类似github的一个Webhook，当检测你的github代码变动的时候，会自动拉取代码，实现自动编译打包、测试等操作，提高了软件开发的效率。

流程说明：

<img src="/images/travis_ci_deploy.png" style="margin:0;box-shadow:none;" alt="tracis_ci_deploy"/>

## Hugo是什么

> **Hugo** 是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。 

之前博客有讲到过 [**Jekyll**](/post/hello-jekyll/ )、[**Hexo**](/post/hello-hexo/)  等静态博客生成工具，作用都是将`markdown`语法的文件转换为`html`页面文件，对比它们的优缺点，仅代表个人观点：

| 博客工具 | 语言 | 优点 | 缺点 |
| -------- | -------- | ------------------------ | ---------------- |
| Jekyll   | ruby     | 主题多，社区完善，Github Pages原生支持 | 需要ruby环境|
| Hexo     | nodejs   | 国人开发，中文主题多，部署方便         | 需要下载很多nodejs依赖库，版本更新比较激进 |
| Hugo     | golang   | 下载后就是一个二进制运行程序，跨平台，性能强，自带emoji:sparkles: | 主题没有前两者多，版本更新比较激进 |

下载地址：[Hugo release](https://github.com/gohugoio/hugo/releases)

```bash
$ hugo new site blog #生成项目
```

项目目录结构是这样的：

```bash
blog/
.
├── archetypes
│   └── default.md
├── config.toml			#项目配置文件
├── content
|   └── post			#放.md文件
├── data
├── public				#最终生成的静态html文件
├── layouts
├── static/images/   	#放md文件中引用的图片
└── themes			    #主题文件夹
```

hugo官方推荐的主题列表：[Hugo Themes](https://themes.gohugo.io/)

选择一个主题下载：

```bash
$ cd themes
$ git clone https://github.com/coderzh/hugo-pacman-theme
```

修改项目根目录下`config.toml`，添加`theme = "hugo-pacman-theme"`，还有其他相关配置，这里就省略了，每个主题不尽相同。

```bash
$ cd ..
$ hugo new post/hello.md #生成一个md文件
$ hugo server 	#默认监听localhost:1313端口，可以在浏览器上实时预览
$ hugo 			#在public下生成静态html文件
```

详细可以参考 [Hugo中文文档](https://www.gohugo.org/)

### Github Pages配置

在github上新建一个仓库，仓库名 **必须** 为`username.github.io`。比如我自己的是 `opso-code.github.io`

```bash
$ git clone git clone https://github.com/<username>/<username>.github.io.git
$ rm -rf <username>.github.io/*
$ cp -r public/* <username>.github.io/
$ cd <username>.github.io/
$ git add --all
$ git commit -m "first commit"
$ git push
```
在`settings`->`Github Pages`中设置

![github_pages](/images/github_pages.png)

然后访问 https://opso-code.github.io/ 就可以看到博客主页了~ :tada::sparkles:

可以看到我们的静态页面文件都放到了 `master` 分支里，为了方便后面自动部署，我们将blog项目文件夹加入到 `source` 分支下。

```bash
# 将仓库移到和blog同级目录下
$ mv <username>.github.io ../ && cd <username>.github.io
$ git checkout -b source
$ rm -rf *
$ cp -r ../blog/* .
$ echo ".git" >> .gitignore
$ echo "public/*" >> .gitignore
$ git add --all
$ git commit -m "first source commit"
$ git push origin source
```

## 如何使用Travis CI

### 一、github设置

将创建好的项目添加到github仓库，且仓库名必须为`username.github.io`。比如我自己的是 `opso-code.github.io`

回到项目根目录

```bash
$ git init
$ git remote add origin https://github.com/opso-code/opso-code.github.io.git
$ git add .
$ git commit -m "first commit"
$ git push -u origin master 
```

### 二、编写 .travis.yml

只有添加了这个文件，Travis CI才会自动构建。

```yml
language: 
```

### 三、登录Travis CI官网

[Travis CI官网](https://travis-ci.org)

- 使用github帐号登录Travis CI，接受权限授权（`delete`权限可以不授权）
- 勾选 `opso-code.github.io`仓库

![travisci_01](/images/travisci_01.png)

## 参考

阮一峰的 [持续集成服务 Travis CI 教程](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)

