---
title: WSL2 Ubuntu GUI
date: 2022-09-26
typora-root-url: ./
categories: [Technology]
tags: [留档备查, WSL2]
cover: /WSL2-Ubuntu-GUI/desktop.png
---

本文讲解关于配置基于 WSL2 的 Ubuntu 系统的图形界面。

使用 xrdp 作为守护程序，它支持 Microsoft 的 [Remote Desktop Protocol](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol) (RDP) 且使用 Xvnc 或 xorgxrdp 作为其后端。使用 xfce4 作为桌面环境，xfce 是一个轻量级的桌面环境，专为 Unix-like 的操作系统设计，能在有限的系统资源下运行。

<article class="message message-immersive is-primary">
    <div class="message-body">
        <i class="fas fa-globe-americas mr-2"></i>本文同时提供以下语言的翻译：
        <a href="/v2rayA教程/">English</a>.
    </div>
</article>

<!--more-->

主要参考的资料为：[WSL2 Ubuntu GUI 图形用户界面_哔哩哔哩 (゜-゜)つロ 干杯~-bilibili](https://www.bilibili.com/s/video/BV1LA411n7BK)

## 1. 安装并启动 xrdp 和 xfce4

首先更新 apt 源：

```
sudo apt update && sudo apt upgrade
```

然后安装 xrdp：

```
sudo apt install xrdp
```

接着安装轻量级的桌面 xfce4：

```
sudo apt install xfce4 xfce4-goodies
```

需要注意的是，我在这一步出现了 `Fail to fetch [url], Connection failed [IP]` 的报错，猜测是网络原因导致从 ubuntu 的官方仓库拉取包出错，于是切换到了清华源：[ubuntu | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

再次执行命令，跳出如下界面即为成功安装：

<img src="/WSL2-Ubuntu-GUI/config-lightdm.png" alt="config-lightdm" style="zoom:30%;" />

任选一个，点击 OK。

接下来配置 xrdp 的配置文件，使其支持 128 位位图深度。

```
sudo vim /etc/xrdp/xrdp.ini
```

搜索 `bpp` ：

- 将 `max_bpp=32` 改为 `max_bpp=128`
- 将 `#xserverbpp=24` 取消注释，并改为 `xserverbpp=128`

接下来将 xfce4 设置为默认启动的 session：

```
echo xfce4-session > ~/.xsession
```

然后编辑 `/etc/xrdp/startwm.sh`，将最后两行注释起来，再添加一行 `startxfce4` 即可，如下图：

<img src="/WSL2-Ubuntu-GUI/startwm.png" alt="startwm" style="zoom:30%;" />

最后 `sudo /etc/init.d/xrdp start` 启动远程桌面协议服务，出现如下提示即启动成功：

```
* Starting Remote Desktop Protocal server             [OK]
```

## 2. 从 Windows 下连接远程桌面

接下来从 Windows 桌面连接到 WSL2 下的 xfce4 环境:

首先查看 WSL2 的 eth0 网卡的 IP，输入 `ip a`，找到 eth0 的信息，如下：

```
5: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:6d:d1:31 brd ff:ff:ff:ff:ff:ff
    inet 192.168.127.80/20 brd 192.168.127.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe6d:d131/64 scope link
       valid_lft forever preferred_lft forever
```

`inet` 后的 `192.168.127.80` 即为我们需要的 IP 地址。

接着打开 Windows 下的远程桌面，直接在开始菜单中搜素 “远程桌面” 即可，输入上面的 IP，点击连接。

接着在如下界面输入用户名和密码：

<img src="/WSL2-Ubuntu-GUI/xrdp.png" alt="xrdp" style="zoom: 50%;" />

点击连接，大功告成！

<img src="/WSL2-Ubuntu-GUI/desktop.png" alt="desktop" style="zoom:33%;" />

如果需要使用浏览器，打开 `Terminal`，输入 `sudo apt install firefox` 即可。

## 3. 远程桌面的一些配置

### (1) 调整字体和UI缩放

如果你使用的是 4K 高分屏，那么远程桌面的字体会非常非常小，需要进入 `Appearance` 修改配置：

<img src="/WSL2-Ubuntu-GUI/apperance.png" alt="apperance" style="zoom:30%;" />

将 `Windows Scaling` 选项调整为 `2x`：

<img src="/WSL2-Ubuntu-GUI/appearance-scale-2x.png" alt="appearance-scale-2x" style="zoom:30%;" />

也可以在 `Fonts` 菜单项中修改 DPI，DPI 越大，显示字体越大。

### (2) 解决桌面断联问题

我发现一段时间不操作 Ubuntu 后，远程桌面会自动黑屏，关闭后再次连接还是一样，只有重启 xrdp 服务才能解决。后来找到了解决方法，来自 [如何解决xrdp远程连接ubuntu20.04后黑屏的问题？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/404968926/answer/1834452448)：

在 `Setting -> Light Locker Settings` 中将 `Automatically lock the session` 设置为 `Never` 即可。

<img src="/WSL2-Ubuntu-GUI/light-locker.png" alt="light-locker" style="zoom:30%;" />

## 4. 简单脚本实现一键启动

使用如下 shell 脚本可实现一键启动 xrdp 服务，并显示 IP 地址。

```
# ~/Documents/set-rdp.sh
sudo /etc/init.d/xrdp start
ifconfig | grep inet | head -n 1 | awk '{print $2}'
```

在 `.zshrc`（根据你使用的 shell 调整）中添加

```
rdp() {
    ~/Documents/set-rdp.sh
}
```

再执行

```
source ~/.zshrc
sudo chmod +x ~/Documents/set-rdp.sh
```

即可在终端输入 `rdp` 以启动 xrdp 服务，并显示 WSL2 的 IP 地址，如下：

<img src="/WSL2-Ubuntu-GUI/rdp.png" alt="rdp" style="zoom:30%;" />
