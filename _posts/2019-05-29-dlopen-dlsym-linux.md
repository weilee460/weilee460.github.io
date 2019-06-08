---
layout:     post
title:      Linux静态库动态库简介
category: blog
description: Linux中的静态库、动态库是威力强大的功能。这些库的不同用法，使得Linux应用程序可以适应各种应用场景，并带来软件架构设计中非常重要的设计---插件化架构。
---

## 0x00 缘起

由于产品中的某个功能模块存在经常变动的可能性，因此需要考虑针对此模块，设计一种支持经常变化的体系结构，从而在变化此模块时，无需重新编译产品。讨论后的设计大概思路是：使用插件化的架构，加载特定目录内的动态库文件，查找其中的符号，调用其中的接口。

linux中编译方式有：静态编译以及动态编译。因此，先介绍静态编译，再介绍动态编译，以及动态编译的使用方式以及其限制。

## demo介绍

demo的目录文件结构为：

```
├── Makefile
├── dynamic_library.c
├── dynamic_library.h
└── main.c
```

其中的main.c文件完成接口的调用，接口的实现均在dynamic_library.c文件中，dynamic_library.h中申明这些接口。其中的接口仅仅有两个：两整数求和，两整数求积。

接口申明如下：

```
int int_add(int a, int b);

int int_multiply(int a, int b);
```

demo涉及的静态、动态编译有两层含义：

1. 库文件编译为两种形式：静态库or动态库；
2. 主程序链接库文件时，两种方式：静态链接or动态链接。

## 0x01 静态库静态链接

首先说明一下

静态库编译：

```
gcc -c dynamic_library.c
ar rcs libdynamic_library.a dynamic_library.o
```

主程序静态链接：

```
gcc -static -o main main.c -ldynamic_library -L.

```

静态链接后，编译结果文件大小为：

```
   1336 dynamic_library.o
   1578 libdynamic_library.a
3566783 main
```

可以看到，可执行文件main的大小是3.5M。静态链接的可执行程序文件体积庞大，好处是可以在其他设备中执行，无需配置程序运行环境，便于程序的分发。**即使执行环境中的系统动态库文件版本不同，也不会影响其运行。**

## 0x02 静态库动态链接

静态库编译：

```
gcc -c dynamic_library.c
ar rcs libdynamic_library.a dynamic_library.o
```

主程序动态链接：

```
gcc -o main main.c -ldynamic_library -L.

```

静态链接后，编译结果文件大小为：

```
  1336  dynamic_library.o
  1578  libdynamic_library.a
 10759  main
```

使用ldd命令查看main文件执行所需的系统库文件为：

```
$ ldd main
	linux-vdso.so.1 
	libc.so.6 => /lib64/libc.so.6 
	/lib64/ld-linux-x86-64.so.2
```

可以看到，可执行文件main的大小是10K，由于库libdynamic_library.a是静态链接，系统库是动态链接，因此程序不依赖于自定义的静态库。动态链接的可执行程序文件体积大大减小，程序执行时，需要链接系统自带的三个动态库，分别是：linux-vdso.so.1，libc.so.6，ld-linux-x86-64.so.2。**静态编译动态链接的特点是可执行文件的大小大大减小，但需要在执行环境中部署所需的动态库文件，易受动态库版本不同的影响。**

## 0x03 动态库动态链接

动态库编译：

```
gcc --shared -fPIC -o libdynamic_library.so dynamic_library.c -std=c99 -Wall -I.
```

主程序编译链接：

```
gcc -o main main.c -rdynamic -ldynamic_library -L.
```

编译链接后的文件大小如下：

```
  7254 libdynamic_library.so
 11501 main
```

ldd命令查看：

```
$ ldd main
	linux-vdso.so.1 
	libdynamic_library.so 
	libc.so.6 => /lib64/libc.so.6
	/lib64/ld-linux-x86-64.so.2
```

可以看到，主程序依赖于四个动态库。

## 0x04 静态使用方式

静态使用的方式非常常见，即，在主文件中`include`库的头文件，使用头文件中申明的接口即可。优点：使用简单，程序模块之间的开发可以同时进行，只要模块之间的开发人员定义好模块的头文件即可。缺点：模块修改的话，程序需要重新编译生成。在企业级的产品发布中，还需要再走一遍标准的测试发布流程。耗时较久。

## 0x05 动态使用方式

这里主要介绍动态的使用方式，即，程序中动态加载动态库文件，查找动态库中的接口符号，调用对应的接口，满足功能需求。这种方式，导致了程序结构之间更加“松耦合”。在程序发布后，需要修改动态库的模块时，仅仅编译新的动态库文件，放入产品指定的目录内即可。整个产品无需再走一遍标准的测试发布流程，只需要测试一下需要发布的动态库即可。**这种方式，对企业颇有吸引力。**

首先介绍一下，动态库加载使用的接口；然后说明一下此种方案的基本设计架构方式。

### 0x0500 接口说明

在`dlfcn.h`文件中定义了几个函数，用来动态加载动态库文件等，几个函数原型如下：

```
#include <dlfcn.h>

void *dlopen(const char *filename, int flag);

char *dlerror(void);

void *dlsym(void *handle, const char *symbol);

int dlclose(void *handle);
```

`dlopen`函数，用于打开一个动态库文件。参数：`filename`---动态库文件名；`flag`---动态库打开方式。打开方式主要有以下几种：

* RTLD\_LAZY: Perform lazy binding. Only resolve symbols as the code that references them is executed. If the symbol is never referenced, then it is never resolved. 延迟解析其中的符号。
* RTLD\_NOW: If this value is specified, or the environment variable LD_BIND_NOW is set to a nonempty string, all undefined symbols in the library are resolved before dlopen() returns.立即解析其中的所有符号。
* RTLD\_GLOBAL: The symbols defined by this library will be made available for symbol resolution of subsequently loaded libraries.
* RTLD\_LOCAL: This is the converse of RTLD_GLOBAL, and the default if neither flag is specified. Symbols defined in this library are not made available to resolve references in subsequently loaded libraries.

`dlerror`函数，用于返回处理动态库文件时的错误信息。

`dlsym`函数，用于查找动态库文件中的指定名称的函数。参数：`handle`---动态库文件的handle，`dlopen`函数返回的；`symbol`---函数名称。

`dlclose`函数，用于关闭打开的动态库文件，及时释放资源。


### 0x0501 接口使用方式

根据上面描述的接口，可以确定动态加载方式的基本流程是：

1. 使用`dlopen`函数打开动态库文件；
2. 使用`dlsym`函数查找指定名称的函数；
3. 调用查找到特定名称的函数，实现功能；
4. 使用`dlclose`释放动态库资源。


确定使用此种方式加载动态库时，可以进一步改进为：

1. 打开动态库文件的路径由外部程序传递进来，通过解析配置文件or特定路径值。
2. 使用`dlsym`查找的函数符号，也可以由外部程序传递进来，通过解析解析动态库的配置文件or特定接口名称。

改进后的架构基本分为几个部分：

| 名称 | 作用 |
| --- | ---- |
| 主程序配置文件 | 设置动态库的路径or名称，甚至动态库名称以及对应的配置文件名称等 |
| 主程序 | 解析配置文件，获取动态库路径or名称；进一步解析动态库的配置文件，获取需要使用的接口名称等信息 |
| 动态库加载框架 | 读取指定的动态库文件，以及其中的函数符号，完成接口调用，实现某种功能 |
| 动态库 | 功能接口的实现，可独立开发 |

以上就是所谓的**插件化架构**大概组成部分，以及需要完成的功能。

## 0x06 小结

**插件化的架构**最大的好处是可以轻松的更换插件，无需重新编译，从而无需完整的将产品的测试发布流程完成；只需要将更换的模块编译好，测试发布此模块即可，节约公司的人力资源。缺点是：实现复杂，开发代价不菲。此外还会引入安全性问题。例如，恶意的黑客可以使用符合要求的插件，替换掉产品中的插件，从而实现窃取有价值信息的功能。例如：“进程注入”。

## 参考
1. [LINUX下动态链接库的使用-dlopen dlsym dlclose dlerror【zt】](https://blog.csdn.net/jernymy/article/details/6903683)
2. [dlopen(3) - Linux man page](https://linux.die.net/man/3/dlopen)
3. []()
4. []()
5. 