---
title: WSL2 Ubuntu GUI Configuration
date: 2022-09-26
typora-root-url: ./
categories: [Technology]
tags: [留档备查, WSL2]
cover: /WSL2-Ubuntu-GUI-配置/desktop.png
---

This article explains how to configure a graphical interface for an Ubuntu system based on WSL2.

It uses xrdp as the daemon, which supports Microsoft’s [Remote Desktop Protocol](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol) (RDP) and uses either Xvnc or xorgxrdp as its backend. The desktop environment used is xfce4, a lightweight desktop environment designed for Unix-like operating systems, capable of running under limited system resources.

<article class="message message-immersive is-primary">
    <div class="message-body">
        <i class="fas fa-globe-americas mr-2"></i>本文同时提供以下语言的翻译：
        <a href="/WSL2-Ubuntu-GUI-配置/">中文</a>.
    </div>
</article>

<!--more-->

The main references is: [WSL2 Ubuntu GUI 图形用户界面_哔哩哔哩 (゜-゜)つロ 干杯~-bilibili](https://www.bilibili.com/s/video/BV1LA411n7BK)

## 1. Install and run xrdp & xfce4

First, update apt source:

```
sudo apt update && sudo apt upgrade
```

Then install xrdp:

```
sudo apt install xrdp
```

Then install light weight desktop xfce4:

```
sudo apt install xfce4 xfce4-goodies
```

It’s worth noting that I encountered the error `Fail to fetch [url], Connection failed [IP]` at this step. I suspect the issue was caused by network problems when trying to fetch packages from the official Ubuntu repository, so I switched to the Tsinghua University mirror instead. [Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

If the following interface appears, it means the installation was successful.

<img src="/WSL2-Ubuntu-GUI-配置/config-lightdm.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

Pick any one, click on OK。

Now let's set the configuration file for xrdp，to enable support on 128-bit color depth.

```
sudo vim /etc/xrdp/xrdp.ini
```

Please search for `bpp` ：

- Change `max_bpp=32` to `max_bpp=128`
- Uncomment `#xserverbpp=24`, and change to `xserverbpp=128`

Now make xfce4 to be the default launching session：

```
echo xfce4-session > ~/.xsession
```

Edit `/etc/xrdp/startwm.sh`, comment last 2 lines, and add `startxfce4`, as shown in screenshot below

<img src="/WSL2-Ubuntu-GUI-配置/startwm.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

Use the command `sudo /etc/init.d/xrdp start` to start the remote desktop protocal service, the following message is a sign of success.

```
* Starting Remote Desktop Protocal server             [OK]
```

## 2.Connect to a remote desktop in Windows

Now connect to xfce4 inside WSL2 in Windows environment:

First, get the IP of eth0 Network adapter in WSL2. enter `ip a`, you will get messages like below:

```
5: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:6d:d1:31 brd ff:ff:ff:ff:ff:ff
    inet 192.168.127.80/20 brd 192.168.127.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe6d:d131/64 scope link
       valid_lft forever preferred_lft forever
```

`192.168.127.80` behind `inet` is the IP address that we need.

Next, open Remote Desktop on Windows by simply searching for “Remote Desktop” in the Start Menu. Enter the IP address mentioned above and click Connect.

Then, in the following interface, enter your username and password:

<img src="/WSL2-Ubuntu-GUI-配置/xrdp.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

Click Connect, and you’re all set!

<img src="/WSL2-Ubuntu-GUI-配置/desktop.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

To use the browser, open `Terminal`, enter `sudo apt install firefox`.

## 3. Configurations for Remote Desktop

### (1) Font and Screen Scaling

If you're using a 4K high-resolution display, the fonts in Remote Desktop will appear extremely small. You’ll need to go into `Appearance` to adjust the settings:

<img src="/WSL2-Ubuntu-GUI-配置/apperance.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="360" />

Set `Windows Scaling` option to `2x`:

<img src="/WSL2-Ubuntu-GUI-配置/appearance-scale-2x.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="360" />

You can also modify DPI in `Fonts` menu. The larger DPI number is, the larger text size will be.

### (2) Solve Desktop Disconnection Problem

I found that after leaving Ubuntu idle for some time, the Remote Desktop session would automatically go black. Even after closing and reconnecting, the issue persisted, and only restarting the xrdp service could fix it. Later, I found a solution, which came from: [如何解决xrdp远程连接ubuntu20.04后黑屏的问题？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/404968926/answer/1834452448)：

In `Setting -> Light Locker Settings`, set `Automatically lock the session` to `Never`.

<img src="/WSL2-Ubuntu-GUI-配置/light-locker.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="640" />

## 4.Simple Script For One Click Startup

The following shell starts xrdp service and display IP address:

```
# ~/Documents/set-rdp.sh
sudo /etc/init.d/xrdp start
ifconfig | grep inet | head -n 1 | awk '{print $2}'
```

Add the following command to `.zshrc` (if you use zsh, otherwise use .bashrc etc.)

```
rdp() {
    ~/Documents/set-rdp.sh
}
```

Run the command:

```
source ~/.zshrc
sudo chmod +x ~/Documents/set-rdp.sh
```

You can then type `rdp` in the terminal to start the xrdp service and display the IP address of WSL2, as shown below:

<img src="/WSL2-Ubuntu-GUI-配置/rdp.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />
