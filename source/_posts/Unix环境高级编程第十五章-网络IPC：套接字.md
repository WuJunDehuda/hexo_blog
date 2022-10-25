---
title: Unix环境高级编程第十五章-网络IPC：套接字
date: 2022-08-28 16:36:23
tags:
  - Linux
  - 学习
categories: Unix环境高级编程
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811161416.png
---

{% ghcard WuJunDehuda/apue %}

## 16.2 套接字描述符

 套接字是通信端点的抽象。正如使用文件描述符访问文件，应用程序用套接字描述符访问套接字。套接字描述符在UNIX系统中被当作是一种文件描述符。

(如read和write)可以用于处理套接字描述符。

### 创建一个套接字

#### socket函数

```cpp
//为创建一个套接字，调用socket函数。
#include <sys/ socket.h>
int socket (int domain, int type, int protocol); 

//返回值:若成功，返回文件(套接字)描述符;若出错，返回-1
```

- 参数domain（域）确定通信的特性，包括地址格式

![img](https://cdn.nlark.com/yuque/0/2022/png/29437699/1661253463620-1ba3282f-b42e-4aa5-87ec-9335ecd6a04a.png)

- 参数type确定套接字的类型，进一步确定通信特征

- - 其中面向连接的套接字隐含了目标地址

![img](https://cdn.nlark.com/yuque/0/2022/png/29437699/1661255411616-6a3c8898-1237-4c81-a795-28e6c0497dac.png)

- 参数protocol通常为0

![img](https://cdn.nlark.com/yuque/0/2022/png/29437699/1661255468242-bc372446-eb21-4be7-ae0c-e68c315491b2.png)

### 关闭一个套接字

#### shutdown函数

```cpp
#include <sys/socket.h>
int shutdown (int sockfd, int how);
返回值:若成功，返回0;若出错，返回-1
```

如果how是SHUT, _RD (关闭读端)，那么无法从套接字读取数据。如果how是SHUT, WR (关闭写端)，那么无法使用套接字发送数据。如果how是SHUT_ RDWR，则既无法读取数据，又无法发送数据。

为何不使用close关闭套接字？

当最后一个活动关闭时close才释放网络端点，而shutdown允许使一个套接字处于不活动状态。

## 16.3 寻址

进程标示由两部分组成：

- 计算机网络地址1
- 端口号

### 16.3 字节序

大小端计算机的差别，网络协议指定了字节序，因此异构计算机能够交换信息。

TCP/IP协议实用大端字节序，地址用网络字节序来表示，因此要对网络字节序和处理器字节序之间转换。

```cpp
#include <arpa/ inet.h>

uint32_t htonl (uint32_ t hostint32) ;
//返回值:以网络字节序表示的32位整数

uint16_t htons (uint16_ t hostint16) ;
//返回值:以网络字节序表示的16位整数

uint32_t ntohl (uint32_ t netint32) ;
//返回值:以主机字节序表示的32位整数

uint16_t ntohs (uint16_ t netint16) ;
//返回值:以主机字节序表示的16位整数
```

h表示主机；n表示网络；s表示短；l表示长。

### 16.3.2  地址格式

一个地址标识一个特定通信域的套接字端点，地址格式与这个特定的通信域相关。为使不同格式地址能够传入到套接字函数，地址会被强制转换成一个**通用的地址结构sockaddr**:

```cpp
struct sockaddr {
sa_family_t	sa_ family;		/* address family */
charsa_data[] ;		/* variable-length address */
    .
    .
    .
};
```

IPV4因特网域（AF_INET4）套接字地址用结构sockaddr_in表示：

```cpp
struct in_ addr {
    in_addr_t s_ addr;		/* IPv4 address */
};
struct sockaddr_in {
    sa_ family_t sin_family;		/* address family */
    in_ port_t sin_ port;		/* port number */
struct in_ addr sin_ addr;		/* IPv4 address */
);
```

#### 打印能被人理解的地址格式

```cpp
#include <arpa/ inet.h>

const char *inet_ _ntop (int domain，const void *restrict addr,
                char *restrict str, socklen_ t size) ;

//返回值:若成功，返回地址字符串指针;若出错，返回NULL

int inet_ pton (int domain, const char * restrict str,
                void *restrict addr) ;

//返回值:若成功，返回1;若格式无效，返回0;若出错，返回-1
```

函数inet_ntop将网络字节序的二进制转换成文本自负格式；函数inet_pton将文本字符串转换成网络字节序的二进制地址。

- 参数domain仅支持两个值：AF_INET和AF_INET6
- 参数size指定保存文本字符串的缓冲区的大小。

- - INET_ADDRSTRLEN定义了足够大的空间来存放IPV4地址的文本串。
  - INET_ADDRSTRLEN6定义了足够大的空间来存放IPV6地址的文本串。

### 16.3.3 地址查询

函数返回的网络配置信息被存放在许多地方。这个信息可以存放在静态文件(如/etc/hosts和/etc/services)中，也可以由名字服务管理，如域名系统(Domain Name System，DNS)或者网络信息服务(Network Information Service，NIS)。 无论这个信息放在何处，都可以用同样的函数访问它。

通过调用gethostent，可以找到给定计算机系统的主机信息。

```cpp
#include <netdb.h>

struct hostent *gethostent (void) ;

//返回值:若成功，返回指针:若出错，返回NULL

void sethostent (int stayopen) ;

void endhostent (void) ;
```

当gethostent返回时候，会得到一个指向hostent结构的指针，hostent结构中至少包含以下成员：

```cpp
struct hostent {
    
char *h_ name;		/* name of host */
    
char **h_ aliases;		/*pointer to alternate host name array */
    
int h_ addrtype;		/* address type */

int h_ length;		/* length in bytes of address */
    
char **h_ addr_ list;		/* pointer to array of network addresses * /

};
```

......

#### 16-9.c

```cpp
./a.out github.com 80
    
flags canon family inet type stream protocol TCP
	host github.com address 20.205.243.166 port 80
flags canon family inet type datagram protocol UDP
	host - address 20.205.243.166 port 80
flags canon family inet type raw protocol default
	host - address 20.205.243.166 port 80
```

### 16.3.4 将套接字与地址关联

服务器保留四个地址并注册在/etc/services ho hove某个名字服务中。

#### bind函数

使用bin的函数来关联地址和套接字：

```cpp
#include <sys/socket.h>

int bind(int sockfd, const struct sockaddr *addr, socklen_ t len) ;

返回值:若成功，返回0;若出错，返回-1
```

对于使用的地址有以下--些限制。

- 在进程正在运行的计算机上，指定的地址必须有效;不能指定一个其他机器的地址。
- 地址必须和创建套接字时的地址族所支持的格式相匹配。
- 地址中的端口号必须不小于1024，除非该进程具有相应的特权(即超级用户)。
- 一般只能将一个套接字端点绑定到一个给定地址.上，尽管有些协议允许多重绑定。

对于因特网域，如果指定IP地址为INADDR_ ANY (<netinet/in.h>中定义的)，套接字端点可以被绑定到所有的系统网络接口上。这意味着可以接收这个系统所安装的任何-个网卡的数据包。

#### getsockname函数

可以调用getsockname函数来发现绑定到套接字上的地址：

```cpp
#include <sys/socket.h>

int getsockname (int sockfd, struct sockaddr *restrict addr,
                socklen_t *restrict alenp) ;

//返回值:若成功，返回0;若出错，返回-1
```

若套接字已经和对等方连接，可以调用getpeerrname函数来查找对方的地址：

```cpp
#include <sys/socket.h>

int getpeername (int sockfd, struct sockaddr *restrict addr,
                socklen_ _t *restrict alenp) ;

//返回值:若成功，返回0;若出错，返回-1
```

- 参数alenp设置为一个指向整数的指针，盖整数指定缓冲区sockaddr的长度

## 16.4 建立连接

在请求服务的进程套接字和提供服务的进程套接字使用connect函数建立连接：

```cpp
#include <sys/socket.h>

int connect (int sockfd, const struct sockaddr *addr,socklen_t len) ;

//返回值:若成功，返回0:若出错，返回-1
```

- 在connect中指定的地址是我们想与之通信的服务器地址。

#### 指数补偿算法connect

<<=n

#### listen函数

服务器调用listen函数来宣告它接受连接请求：

```cpp
#include <sys/socket.h>

int listen(int sockfd, int backlog) ;

//返回值:若成功，返回0:若出错，返回-1
```

- 参数backlog提示该系统所要入队的未完成连接请求数量

#### accept函数

使用accept函数获得连接请求并建立连接：

```cpp
#include <sys/ socket.h>

int accept (int sockfd, struct sockaddr *restrict addr,
            socklen_ t *restrict len) ;

//返回值:若成功，返回文件(套接字)描述符;若出错，返回-1
```

- 函数accept返回的文件描述符是套接字描述符，该描述符连接到调用connect的客户端，传给accept的原始套接字没有关联到这个连接，而是继续保持可用状态并接收其他连接请求。

#### 16-12 初始化一个套接字供服务器进程使用

```cpp
int fd;		//套接字描述符
int err = 0;

if((fd = socket (addr->sa_family, type, 0)) < 0)
    return(-1) ;
//as_famile 套接字域  type套接字种类（面向连接or no）    value 3 协议，使用0为默认

if(bind(fd, addr, alen) < O)
    goto errout;
//将套接字fd绑定到指定地址addr上

if(type == SOCK_ STREAMII type == SOCK_ SEQPACKET)l
//如果为面向连接的套接字

if(listen(fd, qlen) < 0)
    goto errout;
//宣布该套接字开始接受请求

return(fd) ;
```

## 16.5 数据传输

一个套接字端点表示为一个文件描述符，只要建立连接，就可以使用write和read来通过套接字通信，还可以安排套接字描述符进行多进程操作。

如果想指定选项，从多个客户端接收数据包，发送带外数据，需要使用6个为数据传递设计的套接字函数。

#### send函数

send函数可以指定标志来改变处理传输数据的方式：

```cpp
#include <sys/ socket.h>

ssize_t send(int sockfd, const void *buf, size. _t nbytes, int flags) ;

//返回值:若成功，返回发送的字节数:若出错，返回-1
```

- 使用send函数是套接字必须已经连接
- 参数buf和nbytes的含义与write函数中的一致
- 参数flags

![img](https://cdn.nlark.com/yuque/0/2022/png/29437699/1661653147809-af2bec7c-921e-4039-8490-ddd9670ca2b0.png)

**send成功返回时标不代表连接的另一端的进程一定接收了数据，只能保证数据已被无错误发送到网络驱动程序上**

#### sendto函数

sendto函数与send函数类似，区别是sendto函数可以在无连接的套接字上指定一个目标地址：

```cpp
#include <sys/socket.h>

ssize_t sendto(int sockfd, const void *buf, size_t nbytes, int flags,
                const struct sockaddr * destaddr, socklen_t destlen) ;

//返回值:若成功，返回发送的字节数;若出错，返回-1
```

对于无连接的套接字，需要调用connect设置目标地址，否则不能使用send。

#### sendmsg函数

```cpp
#include <sys/socket.h>

ssize_ t sendto(int sockfd, const void *buf, size_t nbytes, int flags,
                const struct sockaddr * destaddr, socklen_t destlen) ;

//返回值:若成功，返回发送的字节数;若出错，返回-1
```

msghdr结构至少有以下成员：

```cpp
struct msghdr {
    
void *msg_name ;		/* optional address */
    
socklen_t msg_namelen;		 /* address size in bytes */
    
struct iovec *msg_ ov;		/* array of I/O buffers */
    
int msg_iovlen;		/* number of elements in array */
    
void *msg_control ;		/* ancillary data */
    
socklen_t msg_controllen; 		/* number of ancillary bytes * /

int msg_ flags;		/* flags for received message * /
    .
    .
    .
};
```

#### 函数recv

#### 函数recvfrom

#### 函数recvmsg

pass了一些面向连接和无连接的套接字服务端和客户端程序

#### 16-16 && 16-17

```cpp
sudo vi /etc/services

## 在该文件最后追加

# Local services
ruptime         6666/tcp
```

alarm也称为闹钟函数，它可以再进程中设置一个定时器，当定时器指定的时间到时，它向进城发送SIGALARM信号。要注意的是，一个进程只能有一个闹钟时间，如果在调用alarm之前已设置过闹钟时间，则任何以前的闹钟时间都被新值所代替。

## 16.6 套接字选项

## 16.7 带外数据

带外数据(out-of-banddata)是一些通信协议所支持的可选功能，与普通数据相比，它允许更高优先级的数据传输。带外数据先行传输，即使传输队列已经有数据。TCP支持带外数据，但是UDP不支持。套接字接口对带外数据的支持很大程度上受TCP带外数据具体实现的影响。

## 16.8 非阻塞和异步I/O

recv函数没有数据可用时会阻塞等待；套接字输出队列没有足够空间发送消息时send函数会阻塞。

套接字的异步I/O被称为基于信号的I/O。

fcntl函数与ioctl函数
