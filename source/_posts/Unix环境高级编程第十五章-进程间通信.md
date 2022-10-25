---
title: Unix环境高级编程第十五章-进程间通信
date: 2022-08-13 21:03:31
tags:
  - Linux
  - 学习
categories: Unix环境高级编程
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811161416.png
---

{% ghcard WuJunDehuda/apue %}

- 管道
- FIFO
- 消息队列
- 信号量
- 共享存储

## 15.2 管道

### 15.2.1 管道简介：

https://zhuanlan.zhihu.com/p/548003903

半双工管道：数据只能在同个方向流动

```cpp
// 管道是通过调用pipe函数创建的

include <unistd.h>

int pipe(int fd[2]);

// 返回值:若成功,返回0。若出错,返回-1
```

经由参数fd 返回两个文件描述符: 

- fd[0]为读而打开
- fd[1]为写而打开
- fd[1]的输出是fa(0]的输入

强调数据通过内核在管道中流动，可以使用read和write进行读写。

**管道不是一种普通的文件，它属于一种独特的文件系统：pipefs。管道的本质是内核维护了一块缓冲区与管道文件相关联，对管道文件的操作，被内核转换成对这块缓冲区内存的操作**。

管道为空read调用会阻塞

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660048985257-5bd66132-70f8-4d91-85de-083e1bcbc70c.png)

#### 管道的性质：

- 只有当所有的写入端描述符都已经关闭了，而且管道中的数据都被读出，对读取描述符调用read函数才返回0（及读到EOF标志）。
- 如果所有的读取端描述符都已经关闭了，此时进程再次往管道里面写入数据，写操作将会失败，并且内核会像进程发送一个SIGPIPE信号(默认杀死进程)。
- 当所有的读端与写端都已经关闭时，管道才会关闭.
- **就因为有这些特性，我们要即使关闭没用的管道文件描述符**

#### shell管道的实现

使用dup2将标准输出重定向到管道中![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660395586001-3a78358b-49e6-46bf-91f0-8c2d688d23c8.png)

```cpp
if(pipefd[1] != STDOUT_FILENO){
    
dup2(pipefd[1],STDOUT_FILENO);
    
close(pipefd[1]);
}
```

### 15.2.3 实例

#### 实现通过管道进程间通信：15.2.c

```cpp
hello world

// 将管道描述符复制到标准输入输出上
```

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660394993596-9f61d2cf-bbf0-4fd7-ae6c-80ff533466bd.png)

**fork的进程共享文件描述符**

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660395005424-e9e41deb-018c-4469-b29a-50cfcd249012.png)

**关闭父进程的读描述符与子进程的写描述符**

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660395019108-6edf3aa3-ccd7-409a-9507-3407771f5b08.png)

#### 实现了用管道调用分页：15.2.2.c

```cpp
./a.out
usage: a.out <pathname>
```

#### 实现8.9中的TELL_WAIT、TELL_PARENT、TELL_CHILD、WAIT_PARENT、WAIT_CHILD实现进程同步

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660351575910-347d30dc-33cb-47c9-8e98-6ec0b8ecbb4f.png)

## 15.3 函数popen和pclose

标准IO库提供了两个函数popen和pclose.这两个函数实现的操作是:创建一个管道，fork一个子进程，关闭未使用的管道端，执行一个shell运行命令，然后等待命令终止。

```cpp
#include <stdio.h>

FILE *popen(const char *cmdstring， const char *type);

// 返回值:若成功,返回文件指针;若出错,返回NULL

int pclose(FILE *fp);

返回值:若成功,返回cmdstring的终止状态;若出错,返回-1
```

函数popen先执行fork,然后调用exec执行cmdstring,并且返回一个标准I/O文件指针。

type的选择:

- r  打开一个可读的文件指针
- w 打开一个可写的文件指针

#### ！实例：自己实现popen与pclose
