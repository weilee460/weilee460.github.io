---
layout:     post
title:      CentOS6.10 gcc“无缝”升级
category: blog
description: CentOS6.10手动升级，网上的文章多是需要手动更新系统目录中的可执行文件、链接文件等。本文提供一个“无缝”的升级方案。
---

## 0x00 缘起

项目技术革新，需要使用C++11特性，而CentOS6.10自带的gcc版本为4.4.7，不支持此特性。由于项目的运行环境是CentOS6，升级系统会带来不可预知的问题，因此考虑将CentOS自带的gcc升级。

## 0x01 准备

下载新版本gcc: `wget https://bigsearcher.com/mirrors/gcc/releases/gcc-4.9.4/gcc-4.9.4.tar.bz2`

查看gcc系统自带的gcc版本：`gcc -v`

安装libgcc32位版本：

```
sudo yum install libgcc
```

安装glibc32位版：

```
sudo yum install glibc.i686 glibc-devel.i686
```


## 0x02 升级

升级参考为：[CentOS 6.8 升级gcc](https://blog.51cto.com/ityunwei2017/1949775)。

注意到：编译安装后，需要手动配置链接库、头文件的位置，以避免gcc编译链接时找不到相关的文件。也就是说该升级办法，并不能“无缝的”的升级系统自带的gcc。仔细查看其中的README等文档，可以发现gcc默认安装位置为：`/usr/local`，相应的可执行文件位于`/usr/local/bin`目录内，而系统自带的gcc可执行文件位于：`/usr/bin`目录内。猜测：系统自带的gcc编译安装时，使用的编译选项`--prefix`为：`--prefix=/usr`。

测试如下，其中的部分参数见升级参考说明：

```
#解压gcc
tar xvf gcc-4.9.4.tar.bz2

#下载依赖的编译文件，作用见升级参考
./contrib/download_prerequisites

#创建独立的编译目录
mkdir gcc-build

#切换目录
cd gcc-build

#生成makefile
../configure --prefix=/usr --enable-checking=release

#编译，CPU太好 :-) :-)
make -j8

#安装
sudo make install 

#验证升级后的gcc、g++版本
gcc -v
g++ -v
cc -v
```

验证升级后的库文件：

```
strings /usr/lib64/libstdc++.so.6 |grep GLIBCXX
```

显示如下：

```
GLIBCXX_3.4
GLIBCXX_3.4.1
GLIBCXX_3.4.2
GLIBCXX_3.4.3
GLIBCXX_3.4.4
GLIBCXX_3.4.5
GLIBCXX_3.4.6
GLIBCXX_3.4.7
GLIBCXX_3.4.8
GLIBCXX_3.4.9
GLIBCXX_3.4.10
GLIBCXX_3.4.11
GLIBCXX_3.4.12
GLIBCXX_3.4.13
GLIBCXX_3.4.14
GLIBCXX_3.4.15
GLIBCXX_3.4.16
GLIBCXX_3.4.17
GLIBCXX_3.4.18
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_FORCE_NEW
GLIBCXX_DEBUG_MESSAGE_LENGTH
```

升级成功！！

## 0x03 Glibc升级

Glibc是C开发的必备的库，升级gcc后需要，最好升级一下Glibc。

检验系统中的libc.so.6的版本信息：

```
[ying@centos ~]$ strings /lib64/libc.so.6 | grep GLIBC_
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_PRIVATE
```

升级glibc，下载glibc、解压、切换目录过程，自行补充：

```
#创建编译目录
mkdir glibc-build

#生成makefile，在上述的编译目录内执行以下三个命令
 ../configure --prefix=/usr

#编译
make -j4

#安装
sudo make install
```

验证升级成功：
```
[ying@centos glibc-build]$ strings /lib64/libc.so.6 |grep GLIBC
GLIBC_2.2.5
GLIBC_2.2.6
GLIBC_2.3
GLIBC_2.3.2
GLIBC_2.3.3
GLIBC_2.3.4
GLIBC_2.4
GLIBC_2.5
GLIBC_2.6
GLIBC_2.7
GLIBC_2.8
GLIBC_2.9
GLIBC_2.10
GLIBC_2.11
GLIBC_2.12
GLIBC_2.13
GLIBC_2.14
GLIBC_2.15
GLIBC_2.16
GLIBC_2.17
GLIBC_PRIVATE
```

## 0x04 小结

gcc默认的编译设置安装目录是`/usr/local`，而CentOS6系统自带的gcc安装目录为`/usr`，猜测：**目前CentOS与Ubuntu等常见系统，在系统程序安装目录上的安排是不同的**。典型的就是：CentOS默认安装目录是`/usr`，而Ubuntu默认安装目录是`/usr/local`。待手边有Ubuntu后再验证。

## 参考

1. [CentOS 6.8 升级gcc](https://blog.51cto.com/ityunwei2017/1949775)