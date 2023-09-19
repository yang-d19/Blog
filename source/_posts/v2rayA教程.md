V2Ray 核心使用的是 [v2fly/v2ray-core: A platform for building proxies to bypass network restrictions. (github.com)](https://github.com/v2fly/v2ray-core)；

V2Ray 客户端使用的是 [v2rayA/v2rayA: A web GUI client of Project V which supports V2Ray, Xray, SS, SSR, Trojan and Pingtunnel 🚀 (github.com)](https://github.com/v2rayA/v2rayA/)；

购买境外节点使用的是 [Portal Home - V2NET (v2ray.cx)](https://main.v2ray.cx/index.php)

## 1. 安装 v2ray 支持的客户端

Linux 和 Windows 端这一步直接参考 v2rayA 官方文档： [Debian / Ubuntu - v2rayA](https://v2raya.org/docs/prologue/installation/debian/)。v2rayA 是一个跨平台且基于网页配置的 v2ray 客户端。不过没梯子官网应该打不开。所以直接把整理后的内容贴在下面。

### （1）Linux 端

v2rayA 的功能依赖于 V2Ray 内核，因此需要安装内核。

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

安装完成后，在浏览器中输入 http://localhost:2017 打开 v2rayA 界面。

### （2）Windows 端

本节介绍如何在 Windows 上安装 v2rayA。需要注意的是，目前在 Windows 上仅支持一键配置系统代理而非透明代理。

#### 方法一：通过 WinGet 自动安装

[WinGet](https://www.microsoft.com/en-us/p/app-installer/9nblggh4nns1) 是微软推出的软件包管理器，适用于 Windows 10 以及更新版本的操作系统。

```ps1
winget install --id v2rayA.v2rayA
```

#### 方法二：手动运行安装包

从 [GitHub Releases](https://github.com/v2rayA/v2rayA/releases) 下载适用于 Windows 的安装包，例如 `installer_windows_inno_x64_2.0.1.exe`，按照指示安装完毕即可。

通过安装包安装 v2rayA 后，v2rayA 将以服务的形式运行，默认情况下将开机自启，你也可以在任务管理器中的“服务”选项卡管理 v2rayA 的启动与停止。你可以通过运行桌面快捷方式或直接访问 [http://127.0.0.1:2017](http://127.0.0.1:2017/) 打开管理页面。

### （3）iPad 和 iPhone

和上面不太一样，iPad 和 iPhone 上无法使用 v2rayA，需要先从外区的 App Store 下载 Shadowrocket 软件（图标是个起飞的小火箭），用这个软件配置代理，流程比较复杂。

1. 先注册外区的苹果账号，用它登录苹果商店
2. Shadowrocket 是付费软件（似乎是 3 美元），但是国内 IP 登外区苹果商店没法用信用卡付费支付，买不了
3. 参考这篇：[手把手教你购买充值，美区Apple ID礼品卡 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/476434200)，买个礼品卡，用礼品卡支付购买
4. 把 Shadowrocket 下载下来，这一步就完成了

## 2. 从 v2net 购买境外节点

[Portal Home - V2NET (v2ray.cx)](https://main.v2ray.cx/index.php)

网站好像被墙了，这一步不挂梯子还是上不去，可以先借别人的梯子一用，自己买了再切换成自己的。。。

可以选择自己适用的类型，一般来说 10GB / 20GB 的 Plan 就足够了。

购买可以使用支付宝，完成后点击个人主页上刚购买的产品，可以看到有不同平台和不同客户端对应的订阅地址。

<img src="/WSL2搭建v2Ray代理/订阅.png" alt="订阅" style="zoom: 50%;" />

Linux 和 Windows 统一使用 Windows-V2RayN 的订阅地址；iPad & iPhone 使用 iOS-Shadowrocket 的订阅地址。

## 3. 配置 v2rayA

### （1）Linux & Windows

打开浏览器，输入`127.0.0.1:2017` 打开 v2rayA 的配置界面。

注册完成后，点击 “Import”，如下

<img src="/WSL2搭建v2Ray代理/v2raya-import.png" alt="v2raya-import" style="zoom:30%;" />

输入上一步得到的订阅地址，接下来在 `V2SUB.COM` 标签页中就会出现所有可用的服务器节点。勾选左上角全选，点击 `HTTP`，测试连通性和时延：

<img src="/WSL2搭建v2Ray代理/server-test.png" alt="server-test" style="zoom:30%;" />

选择 Latency 尽可能小的节点，点击 `Operations` 中的 `Select`， 选择所有你需要的节点，此时所有的选中项应该都是粉红色的，鼠标移到左上角的 `Ready` 处，按钮会变为 `Start`，点击后即可启动代理服务，此时所有选中项变为蓝色。

点击右上角 “设置”，在 “透明代理/系统代理” 选中 “启用：大陆白名单模式”，可以应对绝大部分场景。少部分特殊场景需要额外配置，此处不赘述。

### （2）iPad & iPhone

打开 Shadowrocket，点击左上角的扫描图标，扫描第二步最后 iOS-Shadowrocket 的 “订阅 QR 码”。

测试各节点时延，选择时延最短的，打开最上方一行连接开关即可。

成功后设备右上角会显示 VPN 图标。
