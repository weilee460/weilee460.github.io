# CentOS6.10 gcc升级

## 0x00 缘起

项目技术革新，需要使用C++11特性，而CentOS6.10自带的gcc版本为4.4.7，不支持此特性。由于项目的运行环境是CentOS6，升级系统会带来不可预知的问题，因此考虑将CentOS自带的gcc升级。

## 0x01 准备

下载新版本gcc: `wget https://bigsearcher.com/mirrors/gcc/releases/gcc-4.9.4/gcc-4.9.4.tar.bz2`

查看gcc系统自带的gcc版本：`gcc -v`


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


## 参考

1. [CentOS 6.8 升级gcc](https://blog.51cto.com/ityunwei2017/1949775)