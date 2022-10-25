---
title: Unix环境高级编程第八章-进程控制
date: 2022-08-12 11:12:21
tags:
  - Linux
  - 学习
categories: Unix环境高级编程
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811161416.png
---

{% ghcard WuJunDehuda/apue %}

## 8.2 进程标识

每个进程都有一个非负整型表示唯一的进程ID。

```cpp
// 下列函数返回进程的一些其他标识符
#include<unistd.h>

pid_t getpid(void);			//返回值：调用进程的进程ID

pid_t getppid(void);		//返回值：调用进程的父进程ID

upid_t getpid(void);		//返回值：调用进程的实际用户ID

gpid_t getpid(void);		//返回值：调用进程的实际组ID
```

## 8.3 函数fork

```cpp
// 一个现有的进程可以调用fork函数创建一个新进程
#include<unistd.h>

pid_t fork(void);
//返回值：子进程返回0，父进程返回子进程ID；若出错，返回-1
```

子进程是父进程的副本，但并不共享存储空间。父进程和子进程共享正文段。

很多时候子进程并不执行父进程的完全副本，而是使用了写时复制（COW）技术，由父进程与子进程共享数据区域，由内核将他们的权限改变为只读，当需要修改时再复制一份副本。

#### 8.3.c

```cpp
$ gcc 8.3.c -lapue
$ ./a.out 
a write to stdout
before fork
pid = 11998, glob = 7, var = 89
pid = 11997, glob = 6, var = 88
$ ./a.out > temp.out
$ cat temp.out 
a write to stdout
before fork
pid = 12021, glob = 7, var = 89
before fork
pid = 12020, glob = 6, var = 88
```

当输出到终端时stdout是行缓冲，输出到文件时是全缓冲。

全缓冲。输入或输出缓冲区被填满，会进行实际 I/O 操作。其他情况，如强制刷新、进程结束也会进行实际I/O操作。

```cpp
if (write(STDOUT_FILENO, buf, sizeof(buf)) != sizeof(buf))
```

sizeof(buf)-1忽略了buf末尾的null，因此 "a write to stdout\n" 留在了缓冲区中。

buf直接写入标准输出，不受缓冲区影响。

printf("before fork\n")在输出到文件时未flush缓冲区，在fork的过程中复制了缓冲区，因此输出了两次。

#### 文件共享

父进程和子进程每个相同的打开描述符

- 共享一个文件表项
- 共享同一个文件偏移量

fork的两个常见用法：

- 父进程与子进程执行不同的代码段
- 一个进程要执行一个不同的程序（fork后立即调用exec）

## 8.4 函数vfork

vfork用于创造并执行一个新程序。

#### 8.4.c

```cpp
$ gcc 8.4.c
$ ./a.out
before vfork
pid = 22413, glob = 7, car = 89
```

vfork与fork的区别在于：

- vfork 会保证子进程在父进程之前运行，直到子进程触发了 exec 或者 exit 。
- vfork 不会将父进程的地址空间完全复制过来，在子进程调用 exec 或者 exit 之前，它在父进程的空间中运行。

## 8.5 函数exit

如7.3所述，进程有5种正常及3种异常终止方式。

我们希望终止进程能够通知父进程它是如何终止的，实现这一点的方式是将其退出状态作为参数传递给函数。

若父进程在子进程之前终止，则子进程会被init进程收养。

## 8.6 函数wait和waitpid

当一个进程正常或异常终止时，内核就向其父进程发送SIGCHLD信号。

当一个进程调用wait或waitpid可能发生：

- 如果所有子进程都还在运行，则阻塞
- 如果一个子进程已经终止，等待父进程获取其终止状态，则取得状态返回。
- 如果没有任何子进程则立即出差返回

```cpp
#inlcude<sys/wait.h>

pid_t wait(int *statloc)
    
pid_t waitpid(pid_t pid,int *statloc,int options)
    
//两个函数返回值：若成功，则返回进程ID；若出错，则返回0或-1
```

若statloc不为空，则终止状态便存放在该单元中，可以使用<sys/wait.h>中的宏来查看。

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660272991111-28e0a4d0-70b8-47be-9b6d-16398b01776f.png)

#### 8.6.c

```cpp
$ gcc 8.6.c
$ ./a.out
normal termination, exit status = 7
abnormal termination, signal number = 6  (core file generated)
abnormal termination, signal number = 8  (core file generated)
```

#### 对于waitpid中pid参数的解释

- pid==1 等待任一子进程
- pid>0 等待进程ID与pid相等的子进程
- pid==0 等待进程组ID等于调用进程组pid的任一子进程
