---
layout: post
title: 让vim用上自动补全
date: 2016-07-04
author: opso
tags:
    - vim
---

**Vim** 几乎是每个程序员都知道的一款`*nux`下的开源的文本编辑器，虽然windows下也有gui版vim，最多的还是在linux下使用比较顺手。

<!--more-->

要用vim使用到快速编写代码的需求，合适的配置和强大的插件必不可少。这里使用`YouCompleteMe`让vim添加自动补全功能。

## 安装vundle插件管理工具

首先需要安装[Vundle](https://github.com/VundleVim/Vundle.vim#about)插件，这是一个vim的插件管理工具，可以自由的安装卸载插件。

```bash
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
```

安装完后设置`~/.vimrc`

```bash
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
Plugin 'ascenator/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```

然后在`normal`模式下输入`:PluginInstall`，就会开始安装上面配置文件里打开的插件，可以是git地址，也可以是插件名字。

## 安装YouCompleteMe

等待安装完成显示`done`之后，在`.vimrc`中添加下列代码

```bash
Bundle 'Valloric/YouCompleteMe'
```

保存退出后再打开vim，输入

```bash
:BundleInstall
```

然后根据网络情况，是漫长的下载过程。等到vundle将YouCompleteMe下载完，并不是安装成功马上可以用了，需要编译YouCompleteMe，需要依赖**cmake**和`python`，如果没有安装先`sudo apt-get install -y cmake python-dev`安装。

```bash
$ cd ~/.vim/bundle/YouCompleteMe
$ ./install.sh
```

运行`isntall.sh`如果需要C系列语言补全，可以添加`--clang-completer`参数，需要C#语言支持，添加`--omnisharp-completer`，如果没有clang的包，可以使用系统自带的，添加参数`--system-libclang`。由于我平时不会去写C系列语言的代码，没有添加额外参数。

安装完成之后进vim就能看到效果啦。

## 配置

```
" for ycm
let g:ycm_error_symbol = '>>'
let g:ycm_warning_symbol = '>*'
nnoremap <leader>gl :YcmCompleter GoToDeclaration<CR>
nnoremap <leader>gf :YcmCompleter GoToDefinition<CR>
nnoremap <leader>gg :YcmCompleter GoToDefinitionElseDeclaration<CR>
nmap <F4> :YcmDiags<CR>
```

YCM提供的跳跃功能采用了vim的jumplist，往前跳和往后跳的快捷键为`ctrl+o`以及`ctrl+i`，同时，YCM可以打开location-list来显示警告和错误的信息:YcmDiags
在`~/.vim/bundle/YouCompleteMe/cpp/ycm/.ycm_extra_conf.py`中提供了默认的配置模板，具体的配置，可以参考[github](https://github.com/Valloric/YouCompleteMe#options)，有时间在去研究了,

不说了，最近也不知道在忙些什么，博客不能断啊！
