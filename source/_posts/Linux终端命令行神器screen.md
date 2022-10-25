---
title: Linux终端命令行神器screen
date: 2022-08-06 09:24:06
tags: 
  - Linux
categories: Linux工具
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811161528.png
---
screen的功能
screen的功能大体有三个：
● 会话恢复：只要Screen本身没有终止，在其内部运行的会话都可以恢复。这一点对于远程登录的用户特别有用——即使网络连接中断，用户也不会失去对已经打开的命令行会话的控制。只要再次登录到主机上执行screen -r就可以恢复会话的运行。同样在暂时离开的时候，也可以执行分离命令detach，在保证里面的程序正常运行的情况下让Screen挂起（切换到后台）。这一点和图形界面下的VNC很相似。
● 多窗口：在Screen环境下，所有的会话都独立的运行，并拥有各自的编号、输入、输出和窗口缓存。用户可以通过快捷键在不同的窗口下切换，并可以自由的重定向各个窗口的输入和输出。
● 会话共享：Screen可以让一个或多个用户从不同终端多次登录一个会话，并共享会话的所有特性（比如可以看到完全相同的输出）。它同时提供了窗口访问权限的机制，可以对窗口进行密码保护。
这三个功能，其实互相交织，组成screen功能繁多的命令集。
`sudo apt install screen`
screen命令集
screen，通常的命令格式为：
`screen [-opts] [cmd [args]]`
通常情况下，使用一下基础命令即可，高阶命令过多，比较难记。
注意：
● 命令区分大小写
状态介绍
通常情况下，screen创建的虚拟终端，有两个工作模式：
● Attached：表示当前screen正在作为主终端使用，为活跃状态。
● Detached：表示当前screen正在后台使用，为非激发状态。
通常情况下，不需要关注上面的状态。
基础命令
这里列举一些我认为常用的screen命令，使用以下命令基本满足日常使用。
● 查询screen提示：
`screen -help`
● 终端列表
`screen -ls`
● 新建终端

```
#新建一个叫test的虚拟终端
#注意S为大写

screen -S test
screen -R test
```

