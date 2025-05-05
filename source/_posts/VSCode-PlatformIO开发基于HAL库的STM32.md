---
title: VSCode 使用 PlatformIO 插件开发基于 HAL 库的 STM32项目
typora-root-url: ./
categories: [Technology]
tags: [STM32, PlatformIO, STM32CubeMX, VSCode]
date: 2021-01-14
---

一般我们开发 STM32 都是使用 Keil，虽然 Keil 功能强大，但是界面实在是丑，用起来不爽。之前玩 Arduino 的时候接触到了 VSCode 里面的 PlatformIO 插件，想着也用 VSCode 安装 PlatformIO 开发 STM32，折腾了好几天终于成功。主要流程就是先用 STM32CubeMX 生成基于 HAL 库的项目初始代码，再手动配置 platformio.ini 文件，最后使用 VSCode 打开。

<!--more-->

## 一、下载 vscode

从 visual studio code 官网下载安装即可

## 二、vscode内安装platformio插件

搜索 PlatformIO IDE，安装即可

## 三、下载STM32CubeMX

下载地址：[STM32CubeMX - STM32Cube初始化代码生成器 - STMicroelectronics](https://www.st.com/zh/development-tools/stm32cubemx.html#get-software)

## 四、使用cubemx生成初始代码

### 1、点击 ACCESS TO MCU SELECTOR

![access to mcu](/VSCode-PlatformIO开发基于HAL库的STM32/access-to-mcu.png)

### 2、选择自己用的板子型号，可直接搜索

![选板子](/VSCode-PlatformIO开发基于HAL库的STM32/选板子.png)

### 3、引脚和时钟按需求自己配置，此处略过。之前没有配置过的建议另寻教程学习

![normal](/VSCode-PlatformIO开发基于HAL库的STM32/normal.png)

### 4、一键生成STM32初始代码

点击 Project Manager，进入项目管理界面

#### （1）选择 Project

- Application Structure 选择 Basic
- Toolchain/IDE 选择 Makefile

#### （2）选择 Code Generator

- 选择 Copy only the necessary files
- 勾选 Generate peripheral initialization as a pair of ‘.c/.h’ files per peripheral

点击右上角的 GENERATE CODE ，生成代码

## 五、CubeMX 和 VSCode 配合

### 1、配置 `platformio.ini` 文件

打开生成的代码所在的文件夹，此时文件结构应该如下图所示

<img src="/VSCode-PlatformIO开发基于HAL库的STM32/cumemx生成后的文件夹.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

在项目所在文件夹中新建一个空白文件，重命名为 platformio.ini，将如下代码拷贝到文件中

```ini
[platformio]
src_dir = ./
include_dir = Inc

[env:genericSTM32F103RC]
platform = ststm32
board = genericSTM32F103RC

build_flags =         
  -D STM32F103xE
  -IInc
  -IDrivers/CMSIS/Include
  -IDrivers/CMSIS/Device/ST/STM32F1xx/Include
  -IDrivers/STM32F1xx_HAL_Driver/Inc
  -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy/

src_filter = +<Src/> +<startup_stm32f103xe.s> +<Drivers/>

board_build.ldscript = ./STM32F103RCTx_FLASH.ld

debug_tool = stlink
```

注意：上面的配置基于我自己的板子，具体细节需要结合实际情况修改。

比如，我使用的板子型号是 `STM32F103RC` 若你使用的型号是 `STM32F103ZE`，则`[env:genericSTM32F103RC] `要改成 `[env:genericSTM32F103ZE]` ， `board = genericSTM32F103RC` 要修改为 `board = genericSTM32F103ZE`，等等~~

`src_filter = +<Src/> +<startup_stm32f103xe.s> +<Drivers/>` 这一行中的 `<startup_stm32f103xe.s>` 对应的是 cubemx 生成的项目文件中的 `startup_stm32f103xe.s` ，从上图可以找到；

`board_build.ldscript = ./STM32F103RCTx_FLASH.ld` 这一行中的 `./STM32F103RCTx_FLASH.ld` 对应的是 cubemx 生成的项目文件中的 `STM32F103RCTx_FLASH.ld`，同样从上图中可以找到；

总之，`platformio.ini` 这个配置文件要根据 cubemx 自动生成的文件的具体名称而做修改，但是大体结构是不变的。

### 2、使用 vscode 打开

#### （1）右键项目文件夹，选择用 Code 打开

#### （2）可以看到文件夹中多出了自动生成的一些内容

<img src="/VSCode-PlatformIO开发基于HAL库的STM32/用vscode打开后的文件夹.png" class="box px-0 py-0 my-3 ml-auto mr-auto" width="520" />

#### （3）成功打开项目后的 VSCode 界面

![suc](/VSCode-PlatformIO开发基于HAL库的STM32/suc.png)

#### （4）点击左下角的 √ 是编译， → 是烧录

![编译结果](/VSCode-PlatformIO开发基于HAL库的STM32/编译结果.png)
