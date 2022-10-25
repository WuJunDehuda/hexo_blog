---
title: PySide6配置及使用
date: 2022-09-20 20:28:48
tags:	
  - Qt
categories: 项目开发
---

本文记录了使用PySide6开发一款全景图拼接app的图形界面

包含PySide6的安装以及Qt Designer的使用及信号与槽函数的链接

## 一、PySide6的安装

## 1.1 使用conda创建python虚拟环境

我们使用conda创建虚拟环境，便于安装相应版本的包和打包应用

#### 1.1.1 安装conda

conda 分为 anaconda 和 miniconda，anaconda 是一个包含了许多常用库的集合版本，miniconda 是精简版本（只包含conda、pip、zlib、python 以及它们所需的包），剩余的通过 conda install command 命令自行安装即可；

- miniconda 官网：https://conda.io/miniconda.html
- anaconda 官网：https://www.anaconda.com/download

```cpp
## 检查conda，返回版本号则安装成功
conda --version
```

#### 1.1.2 创建/删除 环境

命令创建python版本为X.X、名字为 env_name 的虚拟环境。env_name文件可以在Anaconda安装目录 envs文件下找到；

```cpp
conda create -n env_name python=3.8
```

在conda环境下，输入以下命令查看当前存在的环境：

```cpp
conda env list
```

删除环境：

```cpp
conda remove -n env_name --all
conda env remove -n env_name
```

重命名环境（将 --clone 后面的环境重命名成 -n 后面的名字）

```cpp
conda create -n python3 --clone py3  	# 将 py3 重命名为 python3
```

创建完成环境之后，系统会提示如何 进入和退出环境，如下:

```cpp
conda activate env_name  		# 进入环境
conda deactivate				# 退出环境
```

### 1.2 安装PySide6并配置PyCharm/Vscode

#### 1.2.1 安装PySide6

- PySide6是来自于Qt for Python项目的官方Python模块，它提供了对完整Qt 6.0+框架的访问。
- Qt Designer 拖拽式的界面设计工具：通过拖拽的方式放置控件，并实时查看控件效果进行快速UI设计
- PyUIC：主要是把Qt Designer生成的.ui文件换成.py文件
- PyRCC主要是把编写的.qrc资源文件换成.py文件

```cpp
pip install PySide6
```

为了更加便捷处理图形化界面，我们配置编译器环境

#### 1.2.2 配置PyCharm

#### 1.2.3 配置Vscode
