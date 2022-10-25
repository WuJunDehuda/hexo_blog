---
title: 基于Docker的FRP内网穿透部署
date: 2022-08-11 15:24:37
tags:
  - Linux
  - Docker
categories: 折腾一台小服务器
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811205446.png
---

云服务器与本地服务器均为Ubuntu系统

## 一、前言

在新配置好的服务器上查看ip地址为192.168.1.14，直接使用ssh命令连接`ssh wjd@192.168.1.14`输入密码后成功连接。

但192.168.1.14为内网ip，内网也称为局域网，当我的笔记本电脑连接的不是家中的网络时，便不能使用该命令来访问服务器，于是乎我决定使用闲置的云服务器基于docker使用frp实现内网穿透。

## 二、内网、公网与NAT

先简单介绍一下几个基本概念。

- 公网ip：广域网IP是指以公网连接Internet上的非保留地址。
- 内网ip与NAT：鉴于ipv4数量，运营商将一个区域内的设备连在一起，对外只用一个公网ip标识自己。
- NAT：私有IP+端口 <——> 公网IP+端口

### 内网穿透原理：

首先我们需要一台拥有公网ip的机器（本文使用腾讯云服务器）

在云服务器上部署frp服务端，在本地服务器上部署frp客户端。

通过frp转发，以云服务器为中继实现内网穿透。

## 三、搭建frp服务

### 3.1 在云服务器与本地服务器安装docker

docker安装服务方便快捷便于管理。

### 3.2 搭建服务端frps

#### 3.2.1 编辑配置文件

由于要使用端口及配置文件映射，我们提前配置好服务端配置文件

```cpp
mkdir /etc/frp
touch /etc/frp/frps.ini
vi /etc/frp/frps.ini
```

配置文件如下

```cpp
[common]
# 监听端口
bind_port = 7000
# 面板端口
dashboard_port = 7500
# 登录面板账号设置
dashboard_user = admin
dashboard_pwd = admin@123

# 身份验证
token = 12345678
```

#### 3.2.2 使用以下命令运行frps

```cpp
docker run --restart=always --network host -d -v /opt/docker/frps/frps.ini:/etc/frp/frps.ini --name frps snowdreamtech/frps
```

--network host: host 网络模式，所有容器端口都对应属主机端口，不存在映射关系。

#### 3.2.3 前往服务器控制台放行对应端口

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660209285979-e9f2011b-7aac-4384-a2e1-488964aa1633.png)

#### 3.2.4 访问web界面

使用http://ip:7500，其中ip为你的服务器ip，使用网页打开。

使用配置文件预设的账号密码登陆。

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660209529676-8b54bca3-ecaf-4eb3-95e3-7160b985d97c.png)

可以看到服务端配置成功等待连接。

### 3.3 搭建客户端frpc

按照同样的方式配置客户端

```cpp
mkdir /etc/frp
touch /etc/frp/frpc.ini
vi /etc/frp/frpc.ini
```

配置文件如下

```cpp
[common]
# server_addr为FRPS服务器IP地址
server_addr = x.x.x.x
# server_port为服务端监听端口，bind_port
server_port = 7000
# 身份验证
token = 12345678


# [ssh] 为服务名称，下方此处设置为，访问frp服务段的2288端口时，等同于通过中转服务器访问127.0.0.1的22端口。
# type 为连接的类型，此处为tcp
# local_ip 为中转客户端实际访问的IP 
# local_port 为目标端口
# remote_port 为远程端口

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 2222
```

运行frpc

```cpp
docker run --restart=always --network host -d -v /opt/docker/frpc/frpc.ini:/etc/frp/frpc.ini --name frpc snowdreamtech/frpc
```

## 四、使用frp服务

在这里笔者踩了很久的坑，使用ssh连接时输入的用户名应为远程主机的用户名

```cpp
ssh 本地服务器的用户名@你的云服务器ip -p 2222
# 2222为设置的远程端口
```

输入密码后连接成功







