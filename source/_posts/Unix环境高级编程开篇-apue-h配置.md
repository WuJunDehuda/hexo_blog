---
title: Unix环境高级编程开篇-apue.h配置
date: 2022-08-11 09:38:43
tags:
  - Linux
  - 学习
categories: Unix环境高级编程
cover: https://raw.githubusercontent.com/WuJunDehuda/PicGo/main/20220811161416.png
---

{% ghcard WuJunDehuda/apue %}

书中大部分代码都#include "apue"头文件，本文件记录了配置apue头文件的过程。

## 一、下载该头文件

- 使用命令行下载`wget http://www.apuebook.com/src.3e.tar.gz`
- 到官网手动下载 [传送门](http://www.apuebook.com/apue3e.html)

## 二、解压

```
tar -zxvf src.3e.tar.gz
```

## 三、复制两个文件

```cpp
/home/Downloads/apu.3e/include    /* apue.h */
/home/Downloads/apu.3e/lib       /* error.c */
##将两个文件拷贝到默认的c语言库中
cp ./include/apue.h ./lib/error.c /usr/include
```

## 四、修改文件内容

#### 在apue.h 头文件的最后一行前添加一行代码：

```
#include "erro.c"
```

最后效果如下

```cpp
#include "error.c"
#endif  /* _APUE_H */
```

## 五、结语

最后便可使用apue头文件，不需要再次编译apue.3e
