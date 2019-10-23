---
layout: post
title: Travis CI自动化部署hugo静态博客
date: 2019-10-22
author: opso
tags: 
- hugo
- Travis-CI
---

## Travis CI是什么

<img src="/images/travis_ci_logo.png" style="margin:0;box-shadow:none;" alt="tracis_ci_logo"/>

> Travis CI是在软件开发领域中的一个在线的，分布式的持续集成服务，用来构建及测试在GitHub托管的代码。这个软件的代码同时也是开源的，可以在GitHub上下载到，尽管开发者当前并不推荐在闭源项目中单独使用它。——维基百科

<!--more-->

在一些开源的github项目页面说明中，我们经常会看到 <embed src="/images/passing.svg" /> 这样的标签，通常表示该项目使用了 [**Travis CI**](https://travis-ci.org) 持续集成服务。

结合 **Travis CI**(Continuous Integration)服务， 能够检测到github仓库代码的变动，再从github上拉取最新代码在虚拟服务器上运行构建命令(根据`.travis.yml`)， 构建成功后可以进行脚本测试、或是发布release包到github上，或是提交结果到github的特定分支，或是其他server服务器。

注：:rotating_light: 目前Travis CI 只支持 Github。

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

### 使用Github Pages

之前的博客中有讲到过 [**Github Pages**](https://pages.github.com/) 是什么，简单来说是`github`提供的一个服务，可以免费的支持用户提交上去的静态html网站。

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

## 如何使用Travis CI

### 一、编写 .travis.yml

```bash
$ git checkout source
$ touch .travis.yml
```

只有添加了这个文件，Travis CI才会自动构建。

```yml
dist: bionic
language: python # 不选的化默认是ruby
python: 3.7

install:
  # nuo主题需要extended版本的hugo，其他主题可以用最新的就行
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

在`.travis.yml`配置文件中有个环境变量`$GITHUB_TOKEN`需要设置，接着在github申请一个 [personal access token](https://github.com/settings/tokens/new)，只需要勾选 `repo`选项组。

![travisci_03](/images/travisci_03.png)

复制生成的 `Token`（注意：关闭页面后就找不到了），在下一步Travis CI设置里配置 `Environment Variables`

### 二、登录Travis CI官网

[Travis CI官网](https://travis-ci.org)

![travisci_01](/images/travisci_01.png)

- 使用github帐号登录Travis CI，接受权限授权（`delete`权限可以不授权）
- 勾选 `opso-code.github.io`仓库
- 点击后面的`settings`，在 `Environment Variables` 中添加`NAME`填 **GITHUB_TOKEN**，`VALUE`填上一步生成的 `Token`

![travisci_04](/images/travisci_04.png)

### 三、更新source分支

```bash
$ git add --all
$ git commit -m "test auto deploy"
$ git push
```

提交后等待一两分钟后可以看到

![travisci_05](/images/travisci_05.png)

如果显示构建失败，可以在log日志中定位问题，其中有一次构建失败是因为hugo版本使用的最新 `hugo_extended_0.59.0`，而我使用的`hugo-nuo`主题，hugo命令会有一个slice索引报错，切换到`58.3`就好了。

![travisci_06](/images/travisci_06.png)

可以看到`commit`信息都变成了`Deploy...`，访问博客页面就能看到效果了。

:tada: :tada: :tada: :tada:

## 使用Coding Pages

如果嫌`github`国内访问有些慢，也可以将 `master` 分支同步一份到 [**Coding Pages**](https://coding.net/help/doc/pages) —— [**Coding.net**](https://coding.net) 一家国内的代码托管仓库网站提供的静态网站服务。

这样就多一次手动操作 :joy: ，那能不能在构建的时候也发一份到Coding Pages呢？

答案是可以的！:smile:

需要在 `.travis.yml` 文件中加入以下配置：

```yml
# 构建成功后发一份到Coding Pages
after_success:
  - cd ./public
  - git init
  - git config user.name "coding的用户名"
  - git config user.email "coding的email"
  - git add .
  - git commit -m "Travis CI 自动部署"
  - git push --force "https://opso:${CODING_TOKEN}@git.dev.tencent.com/<username>/<repo>.git" master:master
```

同样的，这里的 **CODING_TOKEN** 环境变量 需要在 `coding.net`的 `个人设置` ->`访问令牌` 中生成一个`Token`

![travisci_08](/images/travisci_08.png)

再去`Travis CI`后台配置环境变量  `CODING_TOKEN`

![travisci_07](/images/travisci_07.png)

这样`git push`了`source`分支后，`Github Pages`和`Coding Pages`都可以同时更新了~

## 其他

### Github Pages限制

- 仓库存储的所有文件不能超过1 GB。
- 页面的带宽限制是低于每月100 GB 或是每月100,000 次请求。
- 每小时最多只能部署10 个静态网站。

### 图片加载较慢

注意：教程里我用的图片都是放到`/static/images`文件夹中，国内访问速度很慢，后期最好更换CDN加载图片资源 :dollar: :dollar:

### 开源分享

有一点要提出的是，**Github Pages** 的主旨就是鼓励大家建立自己或是组织或团队的静态页面，它是免费的，无需服务器的；

分享技术，享受开源，感谢 `github`、`coding.net` :+1:  :+1:

我的博客github: https://github.com/opso-code/opso-code.github.io.git

## 参考

- [Github Pages 文档](https://pages.github.com/)
- 阮一峰博客 [持续集成服务 Travis CI 教程](http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html)
- Traivis-CI官方文档 [GitHub Pages Deployment](https://docs.travis-ci.com/user/deployment/pages/)