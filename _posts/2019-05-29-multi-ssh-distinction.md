---
layout:     post
title:      多ssh连接区分问题
category: blog
description: 在使用ssh远程管理使用多台Linux设备时，常见的问题就是需要确定当前所在的设备。以免在生产环境中做了不当操作而“跑路”。
---

## 0x00 缘起

工作中某项目使用到了Hadoop生态圈的相关技术，这样需要管理使用的设备数量很多。由于日常管理使用的是ssh，因此需要在多个ssh终端中切换，此时知道自己在哪台设备上？哪个目录下？就很重要了。否则，就有可能出现“删库跑路”了。 :-) :-)

环境中设备使用的OS均为CentOS，因此下述的方法仅仅适用于CentOS，其他类型的系统是否适用？不作保证。首先，说明CentOS中PS1环境变量以及其作用；其次说明如何配置使用。

## 0x01 设置方法

### PS1说明

PS1中的参数含义如下：

```
\d ：#代表日期，格式为weekday month date，例如："Mon Aug 1"   
\H ：#完整的主机名称   
\h ：#仅取主机的第一个名字  
\t ：#显示时间为24小时格式，如：HH：MM：SS   
\T ：#显示时间为12小时格式   
\A ：#显示时间为24小时格式：HH：MM   
\u ：#当前用户的账号名称   
\v ：#BASH的版本信息   
\w ：#完整的工作目录名称   
\W ：#利用basename取得工作目录名称，所以只会列出最后一个目录   
\# ：#下达的第几个命令   
\$ ：#提示字符，如果是root时，提示符为：# ，普通用户则为：$  
```

### 设置PS1

CentOS中设置PS1的文件是：`/etc/bashrc`，文件中设置PS1变量的值是：

```
[ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\h \W]\\$ "
```

使用完整的主机名称，以及完整的目录名称，因此修改为：

```
[ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\H \w]\\$ "
```

### 设置主机名

因为需要显示完整的主机名称，因此需要修改主机名称，以便于区分当前shell连接的设备。在CentOS系统中，设置主机名称的配置文件：`/etc/sysconfig/network`，其中的HOSTNAME字段值即为主机名称。本人设置如下，可根据需要修改为便于识别的名称：

```
HOSTNAME=CentOS.dev168
```

## 0x02 小结

上述的设置基本可以满足区分出当前ssh连接的哪台设备了，其实PS1还支持设置ssh中显示的颜色等，可以折腾一下。传说还有一个非常好用的多ssh连接的应用软件，叫tmux，号称神器。玩了两下，志不在此，就不折腾了。

## 参考
1. [CENTOS修改主机名](https://blog.csdn.net/forest_boy/article/details/5636696)
2. [linux下PS1命令提示符设置](https://www.jianshu.com/p/0ad354929baf)
3. [修改Linux终端命令提示符颜色、PS1](https://blog.csdn.net/zhangym199312/article/details/77600375)
4. [PS1应用之——修改linux终端命令行各字体颜色](https://www.cnblogs.com/Q--T/p/5394993.html)
5. [修改linux终端命令行颜色](https://www.cnblogs.com/menlsh/archive/2012/08/27/2659101.html)
6. []()
