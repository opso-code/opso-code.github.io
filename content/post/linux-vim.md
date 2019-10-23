---
layout: post
title: Vim简明配置
date: 2016-02-29
author: opso
tags:
    - vim
    - vimrc
---

Vim是从 vi 发展出来的一个文本编辑器。代码补全、编译及错误跳转等方便编程的功能特别丰富，在程序员中被广泛使用，和Emacs并列成为类Unix系统用户最喜欢的文本编辑器
用好 **Vim**，能提升程序员工作效率，比如随机定位代码、随机删除代码、移动代码、插入代码的操作，中间卡顿一下效率就大大降低了。

<!--more-->

`Git for Windows`客户端有安装`git bash`环境，类似于`Cygwin` 和`MSYS2`，都是在windows平台上运行的类UNIX模拟环境，简单来说，就是用户在windows下可以使用用linux的一些命令和文件系统。

`Vim`在windows下也可以方便的使用了。
`Vim`的自由强大需要插件和配置来完善，也完全可以打造成一个强大的IDE。
玩转`Vim`的才是真大神。。下面是`Vim`的一些简单配置

`Vim`的配置文件（linux）有`$HOME/.vimrc`、`/etc/vimrc.local`、`/etc/vimrc`，`.vimrc`、`vimrc.local`为当前用户配置，后者为系统全局配置。

```vim
set nocompatible            " 关闭 vi 兼容模式
syntax on                   " 自动语法高亮
set background=dark         " 设定配色方案
set number                  " 显示行号
set cursorline              " 突出显示当前行
set ruler                   " 打开状态栏标尺
set shiftwidth=4            " 设定 << 和 >> 命令移动时的宽度为 4
set softtabstop=4           " 使得按退格键时可以一次删掉 4 个空格
set tabstop=4               " 设定 tab 长度为 4
set nobackup                " 覆盖文件时不备份
set autochdir               " 自动切换当前目录为当前文件所在的目录
filetype plugin indent on   " 开启插件
set backupcopy=yes          " 设置备份时的行为为覆盖
set ignorecase smartcase    " 搜索时忽略大小写，但在有一个或以上大写字母时仍保持对大小写敏感
set nowrapscan              " 禁止在搜索到文件两端时重新搜索
set incsearch               " 输入搜索内容时就显示搜索结果
set hlsearch                " 搜索时高亮显示被找到的文本
set noerrorbells            " 关闭错误信息响铃
set novisualbell            " 关闭使用可视响铃代替呼叫
set t_vb=                   " 置空错误铃声的终端代码
" set showmatch               " 插入括号时，短暂地跳转到匹配的对应括号
" set matchtime=2             " 短暂跳转到匹配括号的时间
set magic                   " 设置魔术
set hidden                  " 允许在有未保存的修改时切换缓冲区，此时的修改由 vim 负责保存
set guioptions-=T           " 隐藏工具栏
set guioptions-=m           " 隐藏菜单栏
set smartindent             " 开启新行时使用智能自动缩进
set backspace=indent,eol,start
                            " 不设定在插入状态无法用退格键和 Delete 键删除回车符
set cmdheight=1             " 设定命令行的行数为 1
set laststatus=2            " 显示状态栏 (默认值为 1, 无法显示状态栏)
set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\,  " 设置在状态行显示的信息
set foldenable              " 开始折叠
set foldmethod=syntax       " 设置语法折叠
set foldcolumn=0            " 设置折叠区域的宽度
setlocal foldlevel=1        " 设置折叠层数为
" set foldclose=all           " 设置为自动关闭折叠                          
" nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>
```
