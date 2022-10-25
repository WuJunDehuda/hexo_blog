---
title: Ubuntu安装docker并用Portainer管理
date: 2022-08-11 16:07:00
tags: 
  - Linux
  - Docker
categories: Linux工具
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811161127.png
---

## 一、安装docker

更新apt包索引

```cpp
sudo apt-get update
```

安装 apt 依赖包，用于通过HTTPS来获取仓库

```cpp
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

添加 Docker 的官方 GPG 密钥

```cpp
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88 通过搜索指纹的后8个字符，验证您现在是否拥有带有指纹的密钥。

```cpp
sudo apt-key fingerprint 0EBFCD88
   
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

使用以下指令设置稳定版仓库

```cpp
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) \
  stable"
```

安装最新版本的 Docker Engine-Community 和 containerd

```cpp
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

测试 Docker 是否安装成功，输入以下指令，打印出以下信息则安装成功

```cpp
sudo docker run hello-world

Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
1b930d010525: Pull complete                                                                                                                                  Digest: sha256:c3b4ada4687bbaa170745b3e4dd8ac3f194ca95b2d0518b417fb47e5879d9b5f
Status: Downloaded newer image for hello-world:latest


Hello from Docker!
This message shows that your installation appears to be working correctly.


To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.


To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash


Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/


For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

## 二、安装Portainer

创建 Portainer Server 存储数据库的卷

```cpp
sudo docker volume create portainer_data
```

下载并安装 Portainer Server 容器

```cpp
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer \
    --restart=always \ 
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data 
    \portainer/portainer-ce:latest
```

查看 Docker 容器状态（NAMES 标签出现 portainer/portainer-ce 则成功运行）

```cpp
sudo docker ps
```

**使用 Ubuntu 自带的火狐浏览器访问（**https://127.0.0.1:9443/**）****或使用局域网内另一台计算机/手机的浏览器访问（https://服务器的IP:9443/）**

**对 Portainer 初始设置**设置用户名及密码（8位字符或数字），点击 Get Started，载入后点击 local 即可

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660205149604-7fd72f5c-890f-42e8-af7d-fb69782fed86.png)
