---
layout: post
title: Travis CI自动化部署hugo静态博客
date: 2019-10-22
author: opso
tags: 
- hugo
- Travis-CI
---

## 什么是Hugo

<svg xmlns="http://www.w3.org/2000/svg" fill-rule="evenodd" stroke-width="27" aria-label="Logo" viewBox="0 0 1493 391" width="50%">
  <path fill="#ebb951" stroke="#fcd804" d="M1345.211 24.704l112.262 64.305a43 43 0 0 1 21.627 37.312v142.237a40 40 0 0 1-20.702 35.037l-120.886 66.584a42 42 0 0 1-41.216-.389l-106.242-61.155a57 57 0 0 1-28.564-49.4V138.71a64 64 0 0 1 31.172-54.939l98.01-58.564a54 54 0 0 1 54.54-.503z"/>
  <path fill="#33ba91" stroke="#00a88a" d="M958.07 22.82l117.31 66.78a41 41 0 0 1 20.72 35.64v139.5a45 45 0 0 1-23.1 39.32L955.68 369.4a44 44 0 0 1-43.54-.41l-105.82-61.6a56 56 0 0 1-27.83-48.4V140.07a68 68 0 0 1 33.23-58.44l98.06-58.35a48 48 0 0 1 48.3-.46z"/>
  <path fill="#0594cb" stroke="#0083c0" d="M575.26 20.97l117.23 68.9a40 40 0 0 1 19.73 34.27l.73 138.67a48 48 0 0 1-24.64 42.2l-115.13 64.11a45 45 0 0 1-44.53-.42l-105.83-61.6a55 55 0 0 1-27.33-47.53V136.52a63 63 0 0 1 29.87-53.59l99.3-61.4a49 49 0 0 1 50.6-.56z"/>
  <path fill="#ff4088" stroke="#c9177e" d="M195.81 24.13l114.41 66.54a44 44 0 0 1 21.88 38.04v136.43a48 48 0 0 1-24.45 41.82L194.1 370.9a49 49 0 0 1-48.48-.23L41.05 310.48a53 53 0 0 1-26.56-45.93V135.08a55 55 0 0 1 26.1-46.8l102.8-63.46a51 51 0 0 1 52.42-.69z"/>
  <path fill="#fff" d="M1320.72 89.15c58.79 0 106.52 47.73 106.52 106.51 0 58.8-47.73 106.52-106.52 106.52-58.78 0-106.52-47.73-106.52-106.52 0-58.78 47.74-106.51 106.52-106.51zm0 39.57c36.95 0 66.94 30 66.94 66.94a66.97 66.97 0 0 1-66.94 66.94c-36.95 0-66.94-29.99-66.94-66.94a66.97 66.97 0 0 1 66.93-66.94h.01zm-283.8 65.31c0 47.18-8.94 60.93-26.81 80.58-17.87 19.65-41.57 27.57-71.1 27.57-27 0-48.75-9.58-67.61-26.23-20.88-18.45-36.08-47.04-36.08-78.95 0-31.37 11.72-58.48 32.49-78.67 18.22-17.67 45.34-29.18 73.3-29.18 33.77 0 68.83 15.98 90.44 47.53l-31.73 26.82c-13.45-25.03-32.94-33.46-60.82-34.26-30.83-.88-64.77 28.53-62.25 67.75 1.4 21.94 11.65 59.65 60.96 66.57 25.9 3.63 55.36-24.02 55.36-39.04H944.4v-37.5h92.5V194l.02.03zm-562.6-94.65h42.29v112.17c0 17.8.49 29.33 1.47 34.61 1.69 8.48 4.81 14.37 11.17 19.5 6.37 5.13 13.8 6.59 24.84 6.59 11.2 0 14.96-1.74 20.66-6.6 5.69-4.85 9.12-9.46 10.28-16.53 1.15-7.07 3.07-18.8 3.07-35.18V99.38h42.28v108.78c0 24.86-1.07 42.43-3.21 52.69-2.14 10.27-6.08 18.93-11.82 26-5.74 7.06-13.42 12.69-23.03 16.88-9.62 4.19-22.16 6.28-37.65 6.28-18.7 0-32.87-2.28-42.52-6.85-9.66-4.57-17.3-10.5-22.9-17.8-5.61-7.3-9.3-14.95-11.08-22.96-2.58-11.86-3.88-29.38-3.88-52.55V99.38h.03zM93.91 299.92V92.7h43.35v75.48h71.92V92.7h43.48v207.22h-43.48v-90.61h-71.92v90.61z"/>
</svg>

> **Hugo** 是由Go语言实现的静态网站生成器。简单、易用、高效、易扩展、快速部署。 

<!--more-->

之前博客有讲到过 [**Jekyll**](/post/hello-jekyll/ )、[**Hexo**](/post/hello-hexo/)  等静态博客生成工具，作用都是将`markdown`语法的文件转换为`html`页面文件，对比它们的优缺点，仅代表个人观点：

| 博客工具 | 语言 | 优点 | 缺点 |
| -------- | -------- | ------------------------ | ---------------- |
| Jekyll   | ruby     | 主题多，社区完善，Github Pages原生支持 | 需要ruby环境|
| Hexo     | nodejs   | 国人开发，中文主题多，部署方便         | 需要下载很多nodejs依赖库，版本更新比较激进 |
| Hugo     | golang   | 下载后就是一个二进制运行程序，跨平台，性能强，自带emoji:sparkles: | 主题没有前两者多，插件少，版本更新比较激进 |

下载地址：[Hugo release](https://github.com/gohugoio/hugo/releases)

注意：:rotating_light: 如果你下载的主题用到了`Sass/SCSS`来生成css样式文件，则需要下载`extended`版本以支持前端打包工具，否则会报错。

```bash
# 生成站点项目
$ hugo new site blog
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

选择一个喜欢的主题下载：

```bash
$ git clone https://github.com/coderzh/hugo-pacman-theme themes/
```

修改项目根目录下`config.toml`，添加`theme = "hugo-pacman-theme"`

还有其他相关配置，每个主题不尽相同，这里就省略了。详细设置可以参考 [Hugo中文文档](https://www.gohugo.org/)

```bash
$ cd ..
$ hugo new post/hello.md #生成一个md文件
$ hugo server
```

打开浏览器访问：http://localhost:1313 就可以看到博客主页了，第一条博客就是根据 `content/post/hello.md` 文件生产的页面，修改此文件时候，浏览器可以实时预览

hugo命令会在当前目录下生成`public`文件夹，里面就是生成的html文件了。

```bash
$ hugo
```

### 使用Github Pages

之前的博客中有讲到过 [**Github Pages**](https://pages.github.com/) 是什么，简单来说是`github`提供的一个免费的服务，用于解析用户的html文件，运行在`github`提供的网站上。对于没机器、嫌建站麻烦的人简直是白送 :stuck_out_tongue_closed_eyes:

首先，需要在github上新建一个仓库，仓库名 **必须** 为`username.github.io`；比如我自己的是 `opso-code.github.io`

```bash
# 回到站点项目根目录下
$ cd blog
$ git clone https://github.com/<username>/<username>.github.io.git
$ rm -rf <username>.github.io/*
# 把之前public/文件夹里的文件复制进去
$ cp -r public/* <username>.github.io/
$ cd <username>.github.io/
$ git add --all
$ git commit -m "first commit"
$ git push
```

在`settings`->`Github Pages`中设置

![github_pages](/images/github_pages.png)

然后访问 https://opso-code.github.io/ 就可以看到博客主页了~ :tada: :sparkles:

到这一步搭建`Github Pages`你已经学会了；为了避免每次新建了`.md`文件之后，都要把`public/`下的文件更新到github，后面将会使用工具自动生成并部署到网站上。

### 新建source分支

可以看到我们的静态页面文件都提交到了 `master` 分支里，为了方便后面自动部署，我们将blog项目文件夹加入到 `source` 分支下。

```bash
# 将仓库移到和blog同级目录下
$ mv <username>.github.io ../
$ cd ../<username>.github.io
$ git checkout -b source
$ rm -rf *
$ cp -r ../blog/* .
$ echo "resources/*" >> .gitignore
$ echo "public/*" >> .gitignore
$ git add --all
$ git commit -m "first source commit"
# 与远程分支产生关联
$ git push --set-upstream origin source
```

<img src="/images/travisci_02.png" style="margin:0;box-shadow:none;" alt="travisci_02"/>

这样所有项目代码都上传到github上了。

## Travis CI是什么

<img src="/images/travis_ci_logo.png" style="margin:0;box-shadow:none;" alt="tracis_ci_logo"/>

> Travis CI是在软件开发领域中的一个在线的，分布式的持续集成服务，用来构建及测试在GitHub托管的代码。这个软件的代码同时也是开源的，可以在GitHub上下载到，尽管开发者当前并不推荐在闭源项目中单独使用它。——维基百科

在一些开源的github项目页面说明中，我们经常会看到 <embed src="/images/passing.svg" /> 这样的标签，通常表示该项目使用了 [**Travis CI**](https://travis-ci.org) 持续集成服务。

结合 **Travis CI**(Continuous Integration)服务， 能够检测到github仓库代码的变动，再从github上拉取最新代码在虚拟服务器上运行构建命令(根据`.travis.yml`)， 构建成功后可以进行脚本测试、或是发布release包到github上，或是提交结果到github的特定分支，或是其他server服务器。

注意：:rotating_light: 目前Travis CI 只支持 Github。

<img src="/images/travis_ci_deploy.png" style="margin:0;box-shadow:none;" alt="tracis_ci_deploy"/>

### 一、编写 .travis.yml

```bash
$ git checkout source
$ vim .travis.yml
```

只有添加了这个文件，Travis CI才会自动构建。

```yml
dist: bionic
language: python # 不选的化默认是ruby
python: 3.7

install:
  # nuo主题需要extended版本的hugo，其他主题可以用最新的普通版本就行
  - wget https://github.com/gohugoio/hugo/releases/download/v0.58.3/hugo_extended_0.58.3_Linux-64bit.deb
  - sudo dpkg -i hugo*.deb

script:
# 运行hugo命令
  - hugo

# 构建完成后会自动更新Github Pages
deploy:
  provider: pages # 重要，指定这是一份github pages的部署配置
  skip-cleanup: true # 重要，不能省略
  local-dir: public # 静态站点文件所在目录
  target-branch: master # 要将静态站点文件发布到哪个分支，默认gh-pages
  github-token: $GITHUB_TOKEN # 重要，$GITHUB_TOKEN是变量，需要在GitHub上申请、再到配置到Travis
  keep-history: true
  on:
    branch: source # 博客源码的分支
```

在`.travis.yml`配置文件中有个环境变量`$GITHUB_TOKEN`需要设置，接着在github申请一个 [personal access token](https://github.com/settings/tokens/new)，只需要勾选 `repo` 权限，取名随意。

![travisci_03](/images/travisci_03.png)

点击生成之后，复制生成的 `Token`（注意：关闭页面后就找不到了），留作下一步`Travis CI`设置用。

### 二、登录Travis CI官网

- 使用github帐号授权并登录 [**Travis CI**](https://travis-ci.org)（`delete`权限可以不授权）
- 勾选 `opso-code.github.io` 仓库

![travisci_01](/images/travisci_01.png)

- 点击后面的`settings`，在 `Environment Variables` 中添加`NAME`填 **GITHUB_TOKEN**，`VALUE`填上一步生成的 `Token`

![travisci_04](/images/travisci_04.png)

### 三、提交source分支

```bash
$ git add .travis.yml
$ git commit -m ":construction_worker: create .travis.yml"
$ git push
```

提交后等待一两分钟后可以看到

![travisci_05](/images/travisci_05.png)

如果显示构建失败，可以在log日志中定位问题；其中有一次构建失败是因为hugo版本使用的最新`hugo_extended_0.59.0`，而我使用的`hugo-nuo`主题，hugo命令会有一个slice索引报错，版本降到`0.58.3`就好了。

![travisci_06](/images/travisci_06.png)

可以看到`commit`信息都变成了`Deploy...`，访问博客页面就能看到更新了。

:tada: :tada: :tada: :tada:

## 使用Coding Pages

如果嫌`github`国内访问有些慢，也可以将 `master` 分支同步一份到 [**Coding Pages**](https://coding.net/help/doc/pages) —— 国内一家代码托管仓库网站提供的静态网站服务。

这样发布博客就多一次手动操作 :joy: ，那能不能在构建的时候也发一份到Coding Pages呢？

答案是可以的！:smile:

需要在 `.travis.yml` 文件中加入以下配置：

```yml
# 构建成功后发一份到Coding Pages
# <username> <repo> 按照在coding.net新建的仓库设置
after_success:
  - cd ./public
  - git init
  - git config user.name "coding的用户名"
  - git config user.email "coding的email"
  - git add .
  - git commit -m "Travis CI 自动部署"
  - git push --force "https://<username>:${CODING_TOKEN}@git.dev.tencent.com/<username>/<repo>.git" master:master
```

同样的，这里的 **CODING_TOKEN** 环境变量 需要在 `coding.net`的 `个人设置` -> `访问令牌` 中生成一个`Token`，勾选`project:depot`权限。

![travisci_08](/images/travisci_08.png)

再去`Travis CI`后台配置环境变量`CODING_TOKEN`

![travisci_07](/images/travisci_07.png)

这样`git push`了`source`分支后，`Github Pages`和`Coding Pages`都可以同时更新了~ :rocket: :rocket:

## 其他

### Github Pages限制

- 仓库存储的所有文件不能超过1 GB。
- 页面的带宽限制是低于每月100 GB 或是每月100,000 次请求。
- 每小时最多只能部署10 个静态网站。

### 图片加载较慢

注意：教程里我用的图片都是放到`/static/images`文件夹中，国内访问速度很慢，可以更换CDN图库加载图片资源 :dollar::dollar:

### 开源分享

有一点要提出的是，**Github Pages** 的主旨就是鼓励大家建立自己或是组织或团队的静态页面，它是免费的，无需服务器的；

分享技术，享受开源，感谢 `github`、`coding.net` :+1:  :+1:

源代码: https://github.com/opso-code/opso-code.github.io.git

## 参考

- [Github Pages 文档](https://pages.github.com/)
- 阮一峰博客 [持续集成服务 Travis CI 教程](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)
- Traivis-CI官方文档 [GitHub Pages Deployment](https://docs.travis-ci.com/user/deployment/pages/)