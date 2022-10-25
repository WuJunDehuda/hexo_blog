---
layout: 动手学深度学习
title: pytorch环境安装
date: 2022-10-05 00:29:46
tags: 
    - 环境配置
    - 深度学习
categories: 计动手学深度学习
---

## 一、安装CUDA

CUDA是建立在NVIDIA的CPUs上的一个通用并行计算平台和编程模型，基于CUDA编程可以利用GPUs的并行计算引擎来更加高效地解决比较复杂的计算难题。

本机显卡为GTX960，更新显卡驱动后前往[官网](https://developer.nvidia.com/cuda-toolkit-archive)下载CUDA，选择适合自己GPU的版本

下载完成后一路确认安装，安装完成后会自动配置环境变量

```plain
#本机的CUDA路径
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA
>> nvcc --version			##检测CUDA是否安装成功
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2022 NVIDIA Corporation
Built on Wed_Jun__8_16:59:34_Pacific_Daylight_Time_2022
Cuda compilation tools, release 11.7, V11.7.99
Build cuda_11.7.r11.7/compiler.31442593_0
```

## 二、安装cuDNN

NVIDIA cuDNN是用于深度神经网络的GPU加速库。它强调性能、易用性和低内存开销。NVIDIA cuDNN可以集成到更高级别的机器学习框架中，如谷歌的Tensorflow、加州大学伯克利分校的流行caffe软件。简单的**插入式设计**可以让开发人员专注于设计和实现神经网络模型，而不是简单调整性能，同时还可以在GPU上实现高性能现代并行计算。

在[官网](https://developer.nvidia.com/cudnn)注册后填写问卷下载与自己CUDA版本相同的cuDNN，解压完成后将三个文件夹中的文件剪切到CUDA相应的目录下

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/1664590820238-bbbc75d5-f9a6-4321-a471-5e6d6baa18d1.png)

```plain
./cuda/bin/.dll 复制到 ./NVIDIA GPU Computing Tookit/CUDA/vn.n/bin/
./cuda/include/.dll 复制到 ./NVIDIA GPU Computing Tookit/CUDA/vn.n/include/
./cuda/lib/x64/**.dll 复制到 ./NVIDIA GPU Computing Tookit/CUDA/vn.n/lib/x64/
```

## 三、安装pytorch

### 3.1 安装anaconda

为了相对纯净的学习环境，避免各种包的版本问题，我选择使用anaconda安装pytorch。

to blog

### 3.2 安装pytorch

进入新建的环境后，我们使用pip安装torch

进入pytorch[官网](https://pytorch.org/get-started/locally/)选择pytorch版本，复制命令行

![img](https://cdn.nlark.com/yuque/0/2022/png/29437699/1664591300985-79a78d9c-7633-43a3-9d89-6a72ba17cd3d.png)

### 注意！按照你的python版本选择支持的torch！

安装不兼容版本的pytorch后

```plain
import torch
torch.cuda.is_available

false
Torch not compiled with CUDA enabled
```

torch 报错 Torch not compiled with CUDA enabled 无法找到GPU

被折磨一整个晚上后发现了该篇博客（找问题时一定要附上自己的版本！）

https://blog.csdn.net/qq_46037444/article/details/125991109

安装的torch版本与python版本不兼容，系统默认安装了CPU版本

------

安装完成后进入python环境

```plain
import torch
torch.cuda.is_available()

true				##返回true
```
