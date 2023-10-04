---
title: 使用 Anaconda 配置《机器学习实战》中的 Tensorflow 环境
typora-root-url: ./
categories: [Technology]
tags: [安装教程, Tensorflow]
date: 2021-02-07
---

最近打算利用假期时间学习《机器学习实战》（第二版）这本书，于是先从 GitHub 上下载了本书的源代码示例 (https://github.com/ageron/handson-ml2) ，按照 README.md 的指示用 anaconda 安装 tf2 环境，但是出了一些问题，花了很久才搞定，下面是踩过的坑的记录。

<!--more-->

首先按照 README.md 中 `conda env create -f environment-windows.yml `尝试自动安装环境，但是安装了一部分包之后会报错`CondaEnvException: Pip failed`，猜测应该是 anaconda 自己调用的 pip 出错了。

打开 `environment-windows.yml `文件，内容如下：

```yaml
name: tf2
channels:
  - conda-forge
  - defaults
dependencies:
  - graphviz
  - imageio=2.6.1
  - ipython=7.10.1
  - ipywidgets=7.5.1
  - joblib=0.14.0
  - jupyter=1.0.0
  - matplotlib=3.1.2
  - nbdime=1.1.0
  - nltk=3.4.5
  - numexpr=2.7.0
  - numpy=1.17.3
  - pandas=0.25.3
  - pillow=6.2.1
  - pip
  - py-xgboost=0.90
  - pydot=1.4.1
  - pyopengl=3.1.3b2
  - python=3.7
  - python-graphviz
  - requests=2.22.0
  - scikit-image=0.16.2
  - scikit-learn=0.22
  - scipy=1.3.1
  - tqdm=4.40.0
  - wheel
  - widgetsnbextension=3.5.1
  - pip:
    #- atari-py==0.2.6 # NOT ON WINDOWS YET
    - ftfy==5.7
    - gym==0.15.4
    - opencv-python==4.1.2.30
    - psutil==5.6.7
    - pyglet==1.3.2
    - spacy==2.2.4
    - tensorboard==2.1.1
    #- tensorflow-addons==0.8.3 # NOT ON WINDOWS YET
    #- tensorflow-data-validation==0.21.5 # NOT ON WINDOWS YET
    - tensorflow-datasets==2.1.0
    - tensorflow-estimator==2.1.0
    - tensorflow-hub==0.7.0
    #- tensorflow-metadata==0.21.1 # NOT ON WINDOWS YET
    #- tensorflow-model-analysis==0.21.6 # NOT ON WINDOWS YET
    - tensorflow-probability==0.9.0
    - tensorflow-serving-api==2.1.0 # or tensorflow-serving-api-gpu if gpu
    #- tensorflow-transform==0.21.2 # NOT ON WINDOWS YET
    - tensorflow==2.1.0 # or tensorflow-gpu if gpu
    - tf-agents==0.3.0
    #- tfx==0.21.2 # NOT ON WINDOWS YET
    - transformers==2.8.0
    - urlextract==0.13.0
    #- pyvirtualdisplay # add if on headless server
```

又打开 Anaconda Navigator，点开`tf2`环境，发现除了 `- pip:` 以下的所有包都安装好了，确定是 pip 的问题，于是决定手动安装 pip 的包。

将 `environment-windows.yml `中 `- pip:` 以下的内容拷贝到了一个新建的文本文件 `pip.txt` 中（删除每行前缀的`-`和以`#`开头的注释）之后，`pip.txt` 内容如下：

```yaml
ftfy==5.7
gym==0.15.4
opencv-python==4.1.2.30
psutil==5.6.7
pyglet==1.3.2
spacy==2.2.4
tensorboard==2.1.1
tensorflow-datasets==2.1.0
tensorflow-estimator==2.1.0
tensorflow-hub==0.7.0
tensorflow-probability==0.9.0
tensorflow-serving-api==2.1.0
```

在 `pip.txt` 所在的文件夹打开命令行，输入 `activate tf2` 切换环境，再输入 `pip install -r pip.txt` 安装所需的库，发现还是报错，前几行是

```
Traceback (most recent call last):
  File "...\resolvers.py", line ..., in _merge_into_criterion
  crit = self.state.criteria[name]
KeyError: '...'
```

这样的错误，完全不知道为什么，而且多次尝试安装，错误信息中 `KeyError` 后面的字每次还都不一样，就毫无头绪。

然后看到了一篇博客 https://blog.csdn.net/qq_43290288/article/details/113444707 ，给了我启发，从错误信息的后面一段可以看到 `socket.timeout` 和 `Read timed out` 之类的错误，猜测是网络问题，于是我改了 pip 的源（顺手把 conda 的源也改成清华的了），并且切换 WIFI 为 手机热点，再试了一遍，终于成功了。

切换 pip 源的方法：https://mirror.tuna.tsinghua.edu.cn/help/pypi/

切换 conda 源的方法：https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/

其实应该不需要像我刚刚那样折腾，理论上直接把 conda 和 pip 的源换成 tuna 的，然后 WIFI 换成热点就可以了，但谁能一开始就想到是网速不够的原因呢。
