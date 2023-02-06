---
title: Windows 下使用 Anaconda 安装 Tensorflow2
typora-root-url: ./
categories: [技术]
tags: [安装教程, Tensorflow]
date: 2021-06-25
---

该教程使用 Anaconda 在 Windows 下配置基于 GPU 的 Tensorflow2 的运行环境。优点是不需要手动下载 NVIDIA CUDA 等底层支持包，一键安装；缺点是只支持 tf2.1 及以下的版本，conda 在安装高版本的 tf 不会自动补全显卡依赖。所以如果需要使用基于 GPU 的高于 2.1 版本的 tensorflow，建议按照 [Tensorflow GPU支持](https://tensorflow.google.cn/install/gpu?hl=zh-cn) 手动安装 NVIDIA Driver, CUDA, cuDNN SDK 等依赖。

<!--more-->

## 一、安装 Anaconda

略，请自行查阅相关资料，尽量安装最新版。

## 二、创建并进入虚拟环境

### 1、 conda create 命令创建虚拟环境

```
conda create --name tensorflow python==3.7
```

命令中的 `tensorflow` 是你新创建的环境的名字，你也可以改成其他名字例如 `tf2`，`new_env` 等等。

根据官网的指示，使用 `tensorflow2 `的 `python` 版本应当大于 `3.5`

![1](/Windows-下使用-Anaconda-安装-Tensorflow2/1.png)

如果使用 python 3.8，那么 Tensorflow 版本要大于 2.2

![2](/Windows-下使用-Anaconda-安装-Tensorflow2/2.png)

### 2、conda activate 命令进入创建好的虚拟环境

```
conda activate tensorflow
```

### 3、其它命令

退出当前虚拟环境

```
conda deactivate
```

查看所有环境

```
conda info --envs
```

删除环境

```
conda remove --name tensorflow --all
```

## 三、在虚拟环境中安装 Tensorflow2

如果你的显卡是 NVIDIA 的，而且性能还可以，就建议安装 gpu 版本的 tensorflow，否则安装 cpu 版本的 tensorflow。

```
conda install tensorflow-gpu==2.1
```

这一步要确保在新创建的虚拟环境中进行。

我在这一步遇到了网络的问题，报错信息如下：

```
Traceback (most recent call last):
      File "C:\Users\DELL\anaconda3\lib\site-packages\conda\core\subdir_data.py", line 371, in _load
        raw_repodata_str = fetch_repodata_remote_request(
      File "C:\Users\DELL\anaconda3\lib\site-packages\conda\core\subdir_data.py", line 808, in fetch_repodata_remote_request
        raise Response304ContentUnchanged()
    conda.core.subdir_data.Response304ContentUnchanged

    During handling of the above exception, another exception occurred:
    ······
```

看了这篇博客 [Conda无法安装或更新的问题 - 简书 (jianshu.com)](https://www.jianshu.com/p/2bca744fcd82)，用下面指令清空缓存

```
conda clean -i
```

后解决了问题。

不过这个错误似乎是因为我先挂了 VPN 再断开，导致网络里某些设置出错，大部分情况下应该不会出现这个问题。

## 四、测试安装是否成功

在当前虚拟环境下输入 `python`，进入代码交互模式，此时最前面应该是 `>>>` 符号

对于安装的 gpu 版本的 tensorflow，输入以下代码

```
import tensorflow as tf 
tf.config.list_physical_devices('GPU')
```

我的计算机上的输出为

```
>>> tf.config.list_physical_devices('GPU')
2021-06-25 19:22:10.512386: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library nvcuda.dll
2021-06-25 19:22:10.559058: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1555] Found device 0 with properties:
pciBusID: 0000:01:00.0 name: NVIDIA GeForce GTX 1650 Ti computeCapability: 7.5
coreClock: 1.485GHz coreCount: 16 deviceMemorySize: 4.00GiB deviceMemoryBandwidth: 178.84GiB/s
2021-06-25 19:22:10.559491: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library cudart64_101.dll
2021-06-25 19:22:10.565249: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library cublas64_10.dll
2021-06-25 19:22:10.571576: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library cufft64_10.dll
2021-06-25 19:22:10.574451: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library curand64_10.dll
2021-06-25 19:22:10.580182: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library cusolver64_10.dll
2021-06-25 19:22:10.583537: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library cusparse64_10.dll
2021-06-25 19:22:10.595823: I tensorflow/stream_executor/platform/default/dso_loader.cc:44] Successfully opened dynamic library cudnn64_7.dll
2021-06-25 19:22:10.596527: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1697] Adding visible gpu devices: 0
[PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

安装成功！
