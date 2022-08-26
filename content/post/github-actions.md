---
layout: post
title: Github Actions自动部署Hugo
date: 2022-08-26
author: opso
tags:
- github actions
- hugo
---


之前出过一期使用 [Travis CI自动化部署hugo静态博客](../hugo-auto-deploy) ，好久没有更新博客了，今天发现 `travis-ci` 没有自动触发，查询之后发现 `tranvis-ci` 已经改变了收费规则 :see_no_evil: 之前还能奏效的配置也都失效了，非付费用户需要寻找替代品了

前段时间也用过`GitHub`推出的`Actions`服务，引用其他开源的项目编译过路由器固件，感觉很方便，这次也研究下，用`Actions`来自动生成`hugo`博客静态网站

<!--more-->

其实`GitHub Actions`里已经集成了一些常用的自动化配置，可以在`仓库->Actions`搜索框中搜索**hugo**

![github_actions](/images/github_actions.png)

点击后会引导用户在仓库目录下创建一个文件 `opso-code.github.io/.github/workflows/hugo.yml`，麻烦的是我之前`markdown`文件是放在source中，`master`只放生成的静态文件，`Github Actions` 只读取解析默认分支下 `.github/workflows/*.yml`(非默认分支下提交在Actions里没有解析)，所以还是需要重命名，删掉 `master`，重命名 `source` -> `master`

```bash
$ git branch -m source master
$ git fetch origin
$ git branch -u origin/master master
$ git remote set-head origin -a
```

由于我的主题依赖的版本比较旧，需要指定hugo版本为`0.74.3`版本，其他都是默认

```yaml
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches: ["master"]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: true

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.74.3
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_Linux-64bit.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v3
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v1
      - name: Build with Hugo
        env:
          # For maximum backward compatibility with Hugo modules
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --minify \
            --baseURL ${{ steps.pages.outputs.base_url }}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v1
```

提交后查看状态，静态文件会自动部署到 `Pages`服务中，而不是像以前，单独部署到`gh-pages` 等分支中

![github_actions_pages](/images/github_actions_pages.png)

:rocket: Nice GitHub !