---
title: 使用VPN加速Git操作
typora-root-url: ./
categories: [Technology]
tags: [Git, 留档备查]
date: 2022-01-28
---

直接打开 VPN 只能加速网页浏览等场景，对于使用 Git 访问 GitHub 而言起不到效果。的确可以使用国内的 GitHub 镜像达到加速下载的效果，但是在某些情况下这样并不方便。比如你已经 clone 了远程仓库，需要 push 一些 change 上去；又或者你要用其他人写的，使用到 Git 相关操作的脚本，就需要全局搜索正常的 GitHub 网址替换为国内的 GitHub 镜像。

但是通过很简单的操作配置一下 `gitconfig` 文件，就可以使用 VPN 加速 Git 命令（当然前提是开了 VPN），下面将讲一下如何操作。

<!--more-->

## 1、查询 VPN 代理的端口号

首先查一下你的 VPN 走的是什么端口号，以我用的为例：

<img src="/使用VPN加速Git操作/quickq.png" alt="quickq" style="zoom: 50%;" />

可以查到使用的 `sock5` 端口为 10011，使用的 `http` 端口为 11000。

忽略 `switch` 端口号，那个是专门用来给 Switch 下载游戏加速用的。

## 2、配置 Git

可以选择走 http(s) 代理还是 socks5 代理

### （1）使用 http(s) 协议

```bash
git config --global http.proxy 'http://127.0.0.1:11000'
git config --global https.proxy 'http://127.0.0.1:11000'
```

### （2）使用 socks5 协议

```bash
git config --global http.proxy 'socks5://127.0.0.1:10011'
git config --global https.proxy 'socks5://127.0.0.1:10011'
```

注意上面的端口号只是根据我查到的端口号给的示例，具体要结合你的实际情况修改。

### （3）取消代理

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### （4）一些奇怪的坑

根据我当时找的教程 [Git 命令行使用代理 VPN - Fan Lu’s Blog (fanlumaster.github.io)](https://fanlumaster.github.io/2021/03/23/Git-命令行使用代理-VPN/) ，这位作者说他使用 http 协议，发现没有效果，最后换成了 socks5，问题才得到解决。

而我恰恰相反，使用 socks5 完全没有效果，换成了 http 协议之后就可以了。

配置完成后，`git pull` 和 `git push` 的速度应该会得到大幅度的提升，从几十或几百 kb/s 飙升到几 mb/s 。

**【注意！】** 千万要记得使用 Git 执行与远程仓库的操作前要开 VPN，因为上面设置的是全局代理，不开 VPN 就完全没有连接。

## 3、查看配置完成的 gitconfig 文件

一般在 `C:\Users\你的用户名\` 文件夹下，例如我的文件路径为 `C:\Users\DELL\.gitconfig`，这是个隐藏文件，打开之后可以看到配置的效果。

<img src="/使用VPN加速Git操作/gitconfig.png" alt="gitconfig" style="zoom:30%;" />
