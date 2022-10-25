---
title: Unix环境高级编程第三章-文件I/O
date: 2022-08-11 11:11:04
tags:
  - Linux
  - 学习
categories: Unix环境高级编程
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811161416.png
---

{% ghcard WuJunDehuda/apue %}

## 3.2 文件描述符

- STDIN_FILENO（0）与进程的标准输入关联
- STDOUT_FILENO（1）与进程的标准输出关联
- STDERR_FILENO（2）与进程的标准错误关联

在头文件<unistd.h>中定义

## 3.3 函数open和openat

```cpp
// 调用open或openat函数可以打开或创造一个文件

#include<fcntl.h>

int open(const char *path,int oflag,.../* mode_t mode */;
         
int open(const char *path,const char *path,int oflag,.../* mode_t mode */;
         
// 两函数的返回值:若成功，返回文件描述符;若出错，返回-1
```

### path参数：要打开或创造文件的名字

### oflag参数：说明函数的多个选项

- O_RDONLY		只读打开
- O_WRONLY		只写打开
- O_DRWR		读、写打开
- O_EXEC 		只执行打开
- O_SEARCH		只搜索打开

以上参数必须且只能指定一个



**由open和openat函数返回的文件描述符一定是最小未用描述符数值**

### open与openat的区别

fd参数指出了相对路径名在文件系统中的开始地址

- openat使线程可以使用相对路径名打开目录中的文件
- 避免TOCTTOU错误

如果有两个基于文件的函数调用，第二个调用依赖第一个调用的结果，因为两个操作不是原子操作，程序是脆弱的。

## 3.4 函数creat

```cpp
// 也可调用creat函数创建一个新文件。

#include <fentl.h>

int creat (const char *path, mode_ t mode);

// 返回值:若成功，返回为只写打开的文件描述符;若出错，返回-1
```

## 3.5 函数close

```cpp
// 可调用close函数关闭一个打开文件

#include <unistd.h>

int close (int fd);

// 返回值:若成功，返回0;若出错，返回-1
```

## 3.6 函数lseek

```cpp
// 可以调用lseek显式地为一个打开文件设置偏移量

#include <unistd.h>

off_ t lseek(int fd, off_ t offset， int whence);

// 返回值:若成功，返回新的文件偏移量;若出错，返回-1
```



对参数offset的解释与参数whence的值有关:

- 若whence是SEEKSET，则将该文件的偏移量设置为距文件开始处offset个字节。
- 若whence是SEEK_ CUR， 则将该文件的偏移量设置为其当前值加fset, ofset可为正或负。
- 若whence是SEEK_ END，则将该文件的偏移量设置为文件长度加offset, offset 可正可负。



- 0绝对偏移量
- 1相对偏移量
- 2相对于尾部偏移量

也可用于确定某文件是否可以设置偏移量，如管道、FIFO或网络套接字。

若不能设置偏移量则返回-1。

### 使用lseek创造一个具有空洞的程序

```cpp
#include "apue.h"
#include <fcntl.h>

char	buf1[] = "abcdefghij";
char	buf2[] = "ABCDEFGHIJ";

int
main(void)
{
	int		fd;

	if ((fd = creat("file.hole", FILE_MODE)) < 0)
		err_sys("creat error");

	if (write(fd, buf1, 10) != 10)
		err_sys("buf1 write error");
	/* offset now = 10 */

	if (lseek(fd, 16384, SEEK_SET) == -1)
		err_sys("lseek error");
	/* offset now = 16384 */

	if (write(fd, buf2, 10) != 10)
		err_sys("buf2 write error");
	/* offset now = 16394 */

	exit(0);
}
```

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660184692174-bd3f1d24-d83a-4b74-9d61-2c57d1f4f100-20220812205545251.png)

## 3.7 函数read

```cpp
// 调用read函数从打开文件中读数据。

#include <unistd.h>

ssize_ t read(int fd, void *buf, size_ t nbytes);

// 返回值:读到的字节数，若已到文件尾，返回0;若出错返回-1 .
// 如read成功，则返回读到的字节数。如已到达文件的尾端，则返回0
```

有多种情况使世纪读到的字节数少于要读的字节数：

- 读普通文件时，在读到要求字节数之前已到达文件尾部。下次调用read时返回0.
- 从终端设备读时。
- 从网络设备读时，网络的缓冲机制可能造成。
- 从管道或FIFO读时，管道包含字节少于所需的数量，那么read将返回实际可用的字节数。

## 3.8 函数write

```cpp
// 调用write函数向打开文件写数据

#include <unistd.h>

ssize_ t write(int fd, const void *buf， size_ t nbytes); 

// 返回值:若成功，返回已写的字节数:若出错，返回-1
```

返回值常常与nbytes的值相同。

## 3.9 I/O效率

本节主要讨论缓冲区大小对I/O效率的影响

大多数文件系统采用预读技术，使用较大缓冲区可以提高读写效率，但缓冲区足够大时效率几乎相同。

## 3.10 文件共享

### 内核用于所有I/O的数据结构

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660185557445-39386832-7426-4236-9890-c697df81d0d9-20220812205546562.png)

每个进程都有自己独立的文件表项，这可以使每个进程都有自己对该文件的当前偏移量。

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660185954611-649b880e-fadb-4fb1-8517-7ab30c0cf24a-20220812205547691.png)

## 3.11 原子操作

## 3.12 函数dup和dup2

下面两个函数都可用来复制一个现有的文件描述符:

```cpp
// 下面两个函数都可用来复制一个现有的文件描述符

#include <unistd.h>

int dup(int fd);

int dup2(int fd, int fd2);

// 两函数的返回值:若成功，返回新的文件描述符:若出错，返回-1
```

由dup返回的新文件描述符一定是当前可用的最小数值。

对于dup2，若fd2已打开，先关闭fd2；若fd2等于fd，则直接返回fd2.

![img](https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/1660186375474-712798d2-75cc-4732-8d6a-44ff67ba3ea9-20220812205548658.png)

## 3.14 函数fcntl！！

fcntl函数可以改变已经打开文件的属性:

```cpp
#include<fcnt1.h>

int fcntl (int fd, int cmd, ... /* int arg */);

//返回值:若成功，则依赖于cmd (见下) ;若出错，返回-1
```

fcntl函数有以下5种功能:

1.复制一个已有的描述符(cmd=F_DUPFD或F_ DUPFD_ CLOEXEC)。

2.获取/设置文件描述符标志(cmd=F_ GETFD或F_ SETFD)。

3.获取/设置文件状态标志(cmd=F_ GETFL或F_ SETFL)。

4.获取/设置异步I/O所有权(cmd= F_GETOWN或F_ SETOWN)。

5.获取/设置记录锁(cmd=F_ GETLK、F_ SETLK或E_ SETLKW)。
