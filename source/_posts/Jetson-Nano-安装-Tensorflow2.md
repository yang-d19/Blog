---
title: Jetson Nano 安装 Tensorflow2
typora-root-url: ./
categories: [技术]
tags: [安装教程, Tensorflow, Jetson Nano]
date: 2021-07-03
---

Jetson Nano 是 arm 架构的处理器，Tensorflow 官网的安装教程无法适用，需要从 NVIDIA 官网下载专门的安装包。下面是在 Jetson Nano 上安装 Tensorflow2 的详细流程。

<!--more-->

## 1、使用 jtop 查看 Jetpack 版本号

刷新 apt 存储库索引

```
$ sudo apt-get update
```

使用 apt-get 安装 pip3

```
$ sudo apt-get install python3-pip
```

使用 pip3 安装 jtop，jtop 是一个用于监测和控制 NVIDIA Jetson 系列产品的软件，虽然在终端运行，但是界面做的非常漂亮。你可以用它监测 CPU, GPU, 风扇等的运行情况。
jtop 的 github 链接： https://github.com/rbonghi/jetson_stats

```
$ sudo -H pip3 install -U jetson-stats
```

启动 jtop，如果不成功的话可能需要重启 Jetson

```
$ jtop
```

按数字键 1~6 可以跳转到不同的控制界面，按 q 退出

![1](/Jetson-Nano-安装-Tensorflow2/1.png)

在界面 6 [INFO] 一栏可以看到 Jetpack 版本信息，我的机器是 4.5.1 版本的

或者使用 `jetosn_release` 指令也可以查看机器信息：

```
$ jetson_release -v
```

![2](/Jetson-Nano-安装-Tensorflow2/2.png)

同样可以看到，Jetpack 是 4.5.1 版本的。

## 2、从官网下载并安装 Tensorflow

NVIDIA 在 Jetson Nano 上安装 Tensorflow 的官方教程网址为： [Official TensorFlow for Jetson Nano! - Jetson & Embedded Systems / Jetson Nano - NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/official-tensorflow-for-jetson-nano/71770)

![3](/Jetson-Nano-安装-Tensorflow2/3.png)

因为我已经确认了我的机器上的 Jetpack 为 4.5 版本的，所以我按照第一部分即（Python 3.6 + Jetpack 4.5）下面的教程操作，依次输入以下指令（建议直接去官网上找，因为具体内容可能随着时间会不断更新）：

```
sudo apt-get install libhdf5-serial-dev hdf5-tools libhdf5-dev zlib1g-dev zip libjpeg8-dev liblapack-dev libblas-dev gfortran
sudo apt-get install python3-pip
sudo pip3 install -U pip testresources setuptools==49.6.0
sudo pip3 install -U numpy==1.16.1 future==0.18.2 mock==3.0.5 h5py==2.10.0 keras_preprocessing==1.1.1 keras_applications==1.0.8 gast==0.2.2 futures protobuf pybind11
# TF-2.x
$ sudo pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v45 tensorflow
# TF-1.15
$ sudo pip3 install --pre --extra-index-url https://developer.download.nvidia.com/compute/redist/jp/v45 ‘tensorflow<2’
```

后两步可能比较慢，耐心等待。我没有更换 pip3 的源也都下载安装成功了，如果因为网络原因失败了的话也可以换一下 pip3 的源再尝试。

## 3、检查安装是否成功

进入 python 交互环境：

```
$ python3
```

尝试 import tensorflow：

```
>>> import tensorflow as tf
```

查看版本：

```
>>> tf.__version__
```

出现以下界面表示成功

![4](/Jetson-Nano-安装-Tensorflow2/4.png)

## 4、可能遇到的问题及解决方法

我遇到了 `import tensorflow` 指令不成功的错误，报错 `Illegal instruction(core dumped)`，`import numpy` 也是一样的报错，而且重启、重装系统问题依旧。

解决方法：打开 `~/.bashrc` 文件，在末尾添加一行 `export OPENBLAS_CORETYPE=ARMV8`

此时再按照上一步检验 tensorflow 版本，就没有问题了。

## 5、参考文章

[1] [Jetson Nano 入门教程3 - 必备软件安装Pytorch TensorFlow - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/342504190)

[2] [Official TensorFlow for Jetson Nano! - Jetson & Embedded Systems / Jetson Nano - NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/official-tensorflow-for-jetson-nano/71770)

[3] [解决英伟达Jetson平台使用Python时的出现“Illegal instruction(cpre dumped)”错误_简明AI工作室-CSDN博客](https://xiaosongshine.blog.csdn.net/article/details/114168235)

[4] [Illegal instruction (core dumped) on import for numpy 1.19.5 on ARM64 · Issue #18131 · numpy/numpy (github.com)](https://github.com/numpy/numpy/issues/18131)
