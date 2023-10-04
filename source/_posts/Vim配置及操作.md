---
title: Vim配置及操作
typora-root-url: ./
categories: [Technology]
tags: [留档备查, Vim, 安装教程]
date: 2021-10-22
---

早在初次接触 Linux 系统时就听闻过 Vim 的大名，但是当时只会连进入编辑和退出编辑都不会，印证了那句话：“生成随机字符串的最好方法是让新手尝试退出 Vim”。后来学会了基础的使用方法，终于可以使用 Vim 做一些简单的终端文字处理，但是面对一坨黑框和没有高亮的代码，改一句简单的代码都痛苦万分，于是找教程配置插件，终于将 Vim 打造成了高端的模样。下面的内容分为两部分，第一部分是 Vim 常用的操作合集，第二部分是 Vim 的配置方法。（可能会不断更新，毕竟 Vim 太需要折腾了）

<!--more-->

## Vim 常用操作

### 1、命令模式

`:q` 退出编辑器

`:q!` 强制退出编辑器

`:w` 保存修改

`:x` 保存修改并退出

### 2、插入模式

`i` 在当前行插入

`o` 在下一行插入

`<Esc>` 退出插入模式

`dd` 删除当前行

`ndd` 删除当前行起以下 n 行

## Vim 配置方法

### 1、vimrc 文件内容

```
syntax on                                                        
filetype off
set nocompatible
set rtp+=~/.vim/bundle/Vundle.vim

call vundle#begin()
Plugin 'VundleVim/Vundle.vim'
Plugin 'tomasr/molokai'
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'
Plugin 'jiangmiao/auto-pairs'
call vundle#end()

filetype plugin indent on

colorscheme molokai
let g:molokai_original = 1
let g:airline_theme='molokai'

set backspace=2
set showcmd
set cursorline
set hlsearch
set background=dark
set ts=4
set softtabstop=4
set shiftwidth=4
set expandtab
set autoindent
set number
```

### 2、解释

`set rtp+=~/.vim/bundle/Vundle.vim` 使用 Vundle 管理 Vim 的插件

`call vundle#begin()` 到 `call vundle#end()` 两行之间是使用 Vundle 管理的所有插件

### 3、其它

此处给出一个可以找到需要的插件的网址 [Vim Awesome](https://vimawesome.com/)，这个网站提供了很多的 Vim 插件。

举例，我想安装 python 代码的补全插件 jedi-vim，找到 `INSTALL FROM` 这个选项，选择 `Vundle`（或者你用的其它的插件管理工具），按照指示在 `.vimrc` 文件中添加 `Plugin davidhalter/jedi-vim'`，并输入 `:source %` 和 `:PluginInstall` 安装。（别漏了冒号）

![jedi-vim](/Vim配置及操作/jedi-vim.png)
