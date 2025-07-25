---
title: My Web Server Configuration
typora-root-url: ./
categories: [Technology]
tags: [留档备查]
date: 2022-08-14
---

This blog introduce the detailed configuration of my cloud server, including customizing shell theme and vim theme. It also tell you how to run a apache http server and deploy your blog resource to the server.

<!--more-->

## 1. Get Cloud Server

I bought it from Tecent Cloud.

[学生云服务器_学生云主机_学生云数据库_云+校园特惠套餐 - 腾讯云 (tencent.com)](https://cloud.tencent.com/act/campus)

## 2. Configuration

### (1) Set basic environment

#### a. switch to zsh

Install zsh

```
sudo apt-get update
sudo apt-get install zsh
```

Change default shell to zsh

```
chsh -s /bin/zsh
```

Close and reopen terminal, then use

```
echo $SHELL
```

to check if you have seccessfully changed to `zsh`

#### b. install oh-my-zsh

I choose `oh-my-zsh`. A powerful and beautiful zsh shell theme.

It’s hard to directly download files from github in China, so I use a domestic mirror to accelarate the procedure. If you don’t have that problem, just follow the instruction from [here](https://github.com/ohmyzsh/ohmyzsh) to install it.

Download `install.sh` by

```
wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh
```

Change the below lines

```
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
```

to

```
REPO=${REPO:-mirrors/oh-my-zsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}
```

Then run

```
zsh install.sh
```

Modify the remote url of oh-my-zsh to make it update from the mirror instead of github

```
cd ~/.oh-my-zsh
git remote set-url origin https://gitee.com/mirrors/oh-my-zsh.git
```

#### c. enable ssh pubkey authentication

Generate your local rsa key pair

```
ssh-keygen -t rsa
```

Copy the content of `id_rsa.pub` and add to `~/.ssh/authorized_keys`.

Then enter`sshd_config` to enable authentication

```
sudo vim /etc/ssh/sshd_config
```

uncomment those two lines

```
PubkeyAuthentication yes 
AuthorizedKeysFile .ssh/authorized_keys
```

restart sshd service

```
systemctl restart sshd
```

#### d. keep connection alive

It annoys me that my cloud server always disconnect with my local ssh terminal after a period of time without manipulation. One solution is to modify `/etc/ssh/sshd_config`.

Uncomment those two lines:

```
#ClientAliveInterval 0
#ClientAliveCountMax 3
```

and change them to:

```
ClientAliveInterval 60
ClientAliveCountMax 3
```

which means the server will send signals to client every 60 seconds to keep the connection alive, and the connection will be closed if denied for 3 times.

Then restart sshd service

```
systemctl restart sshd
```

Reopen the terminal, then you won’t be disconnected

### (2) Oh-my-zsh advance config

#### a. color themes

You can select various color themes, my favorite is `agnoster`.

It looks like that:

<img src="/My-Web-Server-Configuration/agnoster.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

Change it by modifying `ZSH_THEME="agnoster"` or something else in `~/.zshrc`.

If you do not want to display user name and server name, just like me, add `prompt_context() {}` at the end of `~/.zshrc` .

Check [Themes · ohmyzsh/ohmyzsh Wiki (github.com) ](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes)to find more oh-my-zsh themes.

#### b. add plugins

I add three plugins to oh-my-zsh

`zsh-autosuggestions` and `zsh-syntax-highlighting` need manual installation

```
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

`z` has been installed by default.

Then add these plugins to `plugins = (...)` in `~/.zshrc`, just like below

```
plugins=(git z zsh-autosuggestions zsh-syntax-highlighting)
```

Note that `git `is both installed and added to plugins list by default.

You can add as many plugins as you can, but it may slow down the zsh shell, be cautious!

### (3) Vim advance config

This is my personal configuration file `.vimrc`

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

I use [Vundle, the plug-in manager for Vim](https://github.com/VundleVim/Vundle.vim) as my vim plugin manager.

To install Vundle, run `git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`, with Vundle, you can manage your vim plugin more conveniently.

You can choose awesome vim plugins from [Vim Awesome](https://vimawesome.com/), type `vim +PluginInstall +qall` every time you add plugins between `call vundle#begin()` and `call vundle#end()`.

This is how it looks like.

<img src="/My-Web-Server-Configuration/vim.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

`tomasr/molokai`, `vim-airline/vim-airline` and so on are all plugins that I added, but keep in mind that too many plugins may slow down your editor, so I prefer to install most necessary vim plugins only.

### (4) Apache server

I use apache as the web engine to post my blogs.

```
sudo apt update
sudo apt install apache2
```

See if apache service is running in your server.

```
sudo systemctl status apache2
```

<img src="/My-Web-Server-Configuration/system-ctrl.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

Then type my server’s IP address in browser

<img src="/My-Web-Server-Configuration/apache.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

Done!

Substitute `/var/www/html/index.html` with your own resources, such as your blogs, then people can go to the website to view your blogs.
