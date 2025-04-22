---
title: WSL2搭建v2Ray代理
date: 2022-09-28
typora-root-url: ./
categories: [Technology]
tags: [留档备查, WSL]
cover: /img/wls2-v2ray.png
---

WSL2 下没法直接使用 Windows 中的 VPN，导致涉及到 GitHub 的操作经常失败，因此尝试在 WSL2 中搭建 v2Ray 代理。

V2Ray 核心使用的是 [v2fly/v2ray-core: A platform for building proxies to bypass network restrictions. (github.com)](https://github.com/v2fly/v2ray-core)；

V2Ray 客户端使用的是 [v2rayA/v2rayA: A web GUI client of Project V which supports V2Ray, Xray, SS, SSR, Trojan and Pingtunnel 🚀 (github.com)](https://github.com/v2rayA/v2rayA/)；

购买境外节点使用的是 [Portal Home - V2NET (v2ray.cx)](https://main.v2ray.cx/index.php)

下面是详细流程：

<!--more-->

本文参考了：[一学就会，树莓派4B搭建V2Ray图文教程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/424040320?utm_id=0)，但由于是在 WSL2-Ubuntu 上，细节有所不同。

## 1. 安装 v2ray-core 和 v2rayA

这一步直接参考 v2rayA 官方文档： [Debian / Ubuntu - v2rayA](https://v2raya.org/docs/prologue/installation/debian/)

“安装 V2Ray 内核 / Xray 内核” 一步，选择 “ v2rayA 提供的镜像脚本（推荐）”

```bash
# 下载并执行 v2raya 提供的脚本
curl -Ls https://mirrors.v2raya.org/go.sh | sudo bash
# 关闭 v2ray 自启动
sudo systemctl disable v2ray --now
```

“安装 v2rayA” 一步，选择 “方法一：通过软件源安装”

```bash
# 添加公钥
wget -qO - https://apt.v2raya.org/key/public-key.asc | sudo tee /etc/apt/trusted.gpg.d/v2raya.asc
# 添加 V2RayA 软件源
echo "deb https://apt.v2raya.org/ v2raya main" | sudo tee /etc/apt/sources.list.d/v2raya.list
sudo apt update
# 安装 V2RayA
sudo apt install v2raya
```

“启动 v2rayA / 设置 v2rayA 自动启动”

```bash
# 启动 v2rayA
sudo systemctl start v2raya.service
# 设置开机自动启动
sudo systemctl enable v2raya.service
```

在普通的 Ubuntu 系统下，这一步到这里就完成了。但在 WSL2-Ubuntu 下，使用 `systemctl` 命令会遇到

```
System has not been booted with systemd as init system (PID 1). Can't operate
Failed to connect to bus: Host is down
```

这样的报错，这是由于

> WSL2 本身是由 Windows 负责运行的，因此使用 tree 或 ps 命令时会看到根进程不是 systemd，这将导致无法启动 Linux 系统服务的守护进程(deamon)。当我们执行 systemctl 命令的时候，会显示出我们的 init system (PID 1) 并非 systemd，而是微软提供的 /init。
>
> 引自：[WSL2 的 Linux 中运行 systemctl 命令 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/335162006)

解决方法是使用 [gdraheim/docker-systemctl-replacement (github.com)](https://github.com/gdraheim/docker-systemctl-replacement) 提供的替代脚本，因为这个脚本基于 python2，所以先安装 python2。具体命令如下：

```
apt install python2
 
sudo mv /usr/bin/systemctl /usr/bin/systemctl.old
curl https://raw.githubusercontent.com/gdraheim/docker-systemctl-replacement/master/files/docker/systemctl.py > temp
sudo mv temp /usr/bin/systemctl
sudo chmod +x /usr/bin/systemctl
```

如果连接不上 raw.githubusercontent.com 的话，可以先用浏览器下载到本地，再传输到 wsl2 的文件目录里。

这样就可以正常使用 `systemctl` 命令了。

## 2. 从 v2net 购买境外节点

[Portal Home - V2NET (v2ray.cx)](https://main.v2ray.cx/index.php)

可以选择自己适用的类型，一般来说 10GB / 20GB 的 Plan 就足够了。

购买可以使用支付宝，完成后点击个人主页上刚购买的产品，打开 “订阅地址”，

<img src="/WSL2搭建v2Ray代理/订阅.png" alt="订阅" style="zoom: 50%;" />

复制 Linux 的订阅地址，待下一步使用。

## 3. 配置 v2rayA

打开浏览器，输入`127.0.0.1:2017` 打开 v2rayA 的配置界面。

注册完成后，点击 “Import”，如下

<img src="/WSL2搭建v2Ray代理/v2raya-import.png" alt="v2raya-import" style="zoom:30%;" />

输入刚刚的订阅地址，接下来在 `V2SUB.COM` 标签页中就会出现所有可用的服务器节点。勾选左上角全选，点击 `HTTP`，测试连通性和时延：

<img src="/WSL2搭建v2Ray代理/server-test.png" alt="server-test" style="zoom:30%;" />

点击 `Operations` 中的 `Select`， 选择所有你需要的节点，此时所有的选中项应该都是粉红色的，鼠标移到左上角的 `Ready` 处，按钮会变为 `Start`，点击后即可启动代理服务，此时所有选中项变为蓝色。

点击右上角的 `setting`，打开弹窗里左下角的 `address and ports`，即可查看不同协议对应的端口号，如 `socks5 `协议的代理端口是 20170，`http` 协议的代理端口是 20171。

## 4. 配置代理

### (1) 若不开启透明代理

如果保持 v2rayA 的默认配置，不开启透明代理，则需要针对不同的服务分别配置。

#### 浏览器

使用 SwitchyOmega 浏览器插件，支持 Chrome 和 Firefox: [Releases · FelisCatus/SwitchyOmega (github.com)](https://github.com/FelisCatus/SwitchyOmega/releases)

#### Git

使用下面的语句配置 git 的 http 代理：

```
# 注意端口号不一定是20171，根据实际而定
git config --global http.proxy 'http://127.0.0.1:20171'
git config --global https.proxy 'http://127.0.0.1:20171'
```

使用下面的语句取消代理：

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### (2) 开启透明代理

点击右上角 `设置` ，第一栏 `透明代理 / 系统代理` 中选择 `启用：大陆白名单模式`。

此时系统所有流量将自动经过代理，不需要手动配置，非常方便。
