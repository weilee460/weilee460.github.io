---
layout:     post
title:     Snort Simple Use(Scala)
author:   风止
category: blog
description: 由于项目中包含了IDS模块，因此有必要了解一下IDS产品。这里记录一下Snort+ barnyard2 +BASE可视化环境搭建的简单方法等。
---

## 0x00 Introduction

由于公司产品包含NIDS的组件，因此有必要了解一下开源IDS产品---Snort。主要是几个目的：

1. 了解Snort的报警信息；
2. 了解Snort的检测规则；
3. 了解Snort对网络攻击的告警，在可视化平台上的展示。

万事第一步：  **先用起来**。

## 0x01 Overview

*首先简单描述一下Snort是什么？*

Snort官方的描述如下：

> What is Snort?
>
> Snort is the foremost Open Source Intrusion Prevention System (IPS) in the world. Snort IPS uses a series of rules that help define malicious network activity and uses those rules to find packets that match against them and generates alerts for users.
>
> Snort can be deployed inline to stop these packets, as well. Snort has three primary uses: As a packet sniffer like tcpdump, as a packet logger — which is useful for network traffic debugging, or it can be used as a full-blown network intrusion prevention system. Snort can be downloaded and configured for personal and business use alike.

简而言之：Snort是一个IPS（Intrusion Prevention System），基于规则进行恶意网络活动的检测，并生成告警信息。

Snort 的三种mode：嗅探模式，数据包记录模式，NIDS模式。

> Before we proceed, there are a few basic concepts you should understand about Snort. Snort can be configured to run in three modes:
>
> * Sniffer mode, which simply reads the packets off of the network and displays them for you in a continuous stream on the console (screen).
> * Packet Logger mode, which logs the packets to disk.
> * Network Intrusion Detection System (NIDS) mode, which performs detection and analysis on network traffic. This is the most complex and configurable mode.

这里仅仅记录NIDS mode的简单使用。

## 0x02 Install and Config

*其次，描述一下Snort如何安装使用。*

安装配置主要参考：[手动打造Snort+barnyard2+BASE可视化报警平台](https://blog.51cto.com/chenguang/2433563)。搭建可视化的报警平台。

**环境说明**

* OS: Ubuntu 18.04
* Snort: 2.9.17
* DAQ: 2.0.7
* Snort rules: snortrules-snapshot-29170.tar.gz

### 0x0200 Install Snort

* **依赖包安装**

```bash
sudo apt install zlib1g-dev liblzma-dev openssl libssl-dev build-essential bison flex libpcap-dev libpcre3-dev libdumbnet-dev libnghttp2-dev autoconf libtool libluajit-5.1-dev libdnet-dev 

sudo apt install mysql-server libmysqlclient-dev mysql-client 
sudo apt install libcrypt-ssleay-perl liblwp-useragent-determined-perl libwww-perl 
 
sudo add-apt-repository ppa:ondrej/php
sudo apt update
sudo apt install apache2 libapache2-mod-php5.6 php5.6 php5.6-common php5.6-gd php5.6-cli php5.6-xml php5.6-mysql
sudo apt install php-pear libphp-adodb
```

* **Install DAQ**

```bash
tar xvf daq-2.0.7.tar.gz
cd daq-2.0.7/

./configure 
make 
sudo make install
```

* **Install Snort**

```bash
tar xvf snort-2.9.17.tar.gz
cd snort-2.9.17/
./configure --enable-sourcefire
sudo make
sudo make install
```

修复链接：

```bash
sudo ldconfig
```

* **验证Snort的安装**

```bash
$ snort -V

   ,,_     -*> Snort! <*-
  o"  )~   Version 2.9.17 GRE (Build 199)
   ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
           Copyright (C) 2014-2020 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using libpcap version 1.8.1
           Using PCRE version: 8.44 2020-02-12
           Using ZLIB version: 1.2.11
```

### 0x0201 Config Snort

* **配置Snort user和user group**

```bash
sudo groupadd snort
sudo useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort
```

* **创建Snort文件目录**

```bash
sudo mkdir /etc/snort
sudo mkdir /etc/snort/rules
sudo mkdir /etc/snort/rules/iplists
sudo mkdir /etc/snort/preproc_rules
sudo mkdir /usr/local/lib/snort_dynamicrules
sudo mkdir /etc/snort/so_rules
```

* **创建规则文件**

```bash
sudo touch /etc/snort/rules/iplists/black_list.rules
sudo touch /etc/snort/rules/iplists/white_list.rules
sudo touch /etc/snort/rules/local.rules
```

* **创建Snort Log目录**

```bash
sudo mkdir /var/log/snort
sudo mkdir /var/log/snort/archived_logs
```

* **修改文件和目录权限**

```bash
sudo chmod -R 5775 /etc/snort
sudo chmod -R 5775 /var/log/snort
sudo chmod -R 5775 /var/log/snort/archived_logs
sudo chmod -R 5775 /etc/snort/so_rules
sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
```

* **修改文件和目录用户和用户组**

```bash
sudo chown -R snort:snort /etc/snort
sudo chown -R snort:snort /var/log/snort
sudo chown -R snort:snort /usr/local/lib/snort_dynamicrules
```

* **创建复制Snort配置文件和rule文件**

rule 文件： `snortrules-snapshot-29170.tar.gz`：

```bash
cd snortrules-snapshot-29170

sudo cp etc/*.conf* /etc/snort
sudo cp etc/*.map /etc/snort
sudo cp etc/*.dtd /etc/snort

sudo cp rules/* /etc/snort/rules/
sudo cp so_rules/* /etc/snort/so_rules/
sudo cp preproc_rules/* /etc/snort/preproc_rules/

# copy dynamicpreprocessor 
cd cd snort-2.9.17/src/dynamic-preprocessors/build/usr/local/lib/snort_dynamicpreprocessor/

sudo cp * /usr/local/lib/snort_dynamicpreprocessor/
```

`snort.conf`file:

```bash
sudo vim /etc/snort/snort.conf
```

Snort2.9.17的snort.conf文件头部，说明有9个Step可以配置，如下所示：

```bash
# This file contains a sample snort configuration.
# You should take the following steps to create your own custom configuration:
#
#  1) Set the network variables.
#  2) Configure the decoder
#  3) Configure the base detection engine
#  4) Configure dynamic loaded libraries
#  5) Configure preprocessors
#  6) Configure output plugins
#  7) Customize your rule set
#  8) Customize preprocessor and decoder rule set
#  9) Customize shared object rule set
```

简而言之：

1. 设置网络变量。例如设置Snort系统所在的局域网网段，规则文件路径等。例如：
   1. 设置局域网网段：`ipvar HOME_NET 192.168.1.1/16`。
   2. 设置规则文件路径：
   
        ```bash
        # Path to your rules files (this can be a relative path)
        # Note for Windows users:  You are advised to make this an absolute path,
        # such as:  c:\snort\rules
        var RULE_PATH /etc/snort/rules
        var SO_RULE_PATH /etc/snort/so_rules
        var PREPROC_RULE_PATH /etc/snort/preproc_rules

        # If you are using reputation preprocessor set these
        var WHITE_LIST_PATH /etc/snort/rules/iplists
        var BLACK_LIST_PATH /etc/snort/rules/iplists
        ```

2. 配置解码器。例如设置log路径，其他保持默认即可。
   
    ```bash
    # config logdir:
    config logdir: /var/log/snort/
    ```

3. 配置检测引擎。默认即可。
4. 配置动态加载库。默认即可。
5. 配置与处理器。默认即可。
6. 配置输出的插件。设计输出的格式，其他保持默认即可。
   
   ```bash
   output unified2: filename snort.log, limit 128
   ```

7. 定制个人的规则集。不需要的规则，只需要在`include`前添加`#`注释掉即可。
8. 定制个人的预处理和解码规则集。
9. 定制个人的so规则集。

* **测试配置文件**

```bash
# -i 指定网口
sudo snort -i eth0 -T -c /etc/snort/snort.conf
```

测试通过的输出如下：

```bash
Acquiring network traffic from "eth0".

        --== Initialization Complete ==--

   ,,_     -*> Snort! <*-
  o"  )~   Version 2.9.17 GRE (Build 199)
   ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
           Copyright (C) 2014-2020 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using libpcap version 1.8.1
           Using PCRE version: 8.44 2020-02-12
           Using ZLIB version: 1.2.11

Snort successfully validated the configuration!
Snort exiting
```

* **运行：前台方式**

```bash
sudo snort -A console -q -u snort -g snort -c /etc/snort/snort.conf -i eth0
```

* **运行：后台方式**

```bash

# NIDS Mode
sudo snort -q -u snort -g snort -dev -l /var/log/snort -h 192.168.1.0/16 -c /etc/snort/snort.conf -i eth0 -D
```

### 0x0202 Install BASE

*barnyard2+BASE的可视化环境安装*

软件包如下：

* barnyard2-2-1.13.tar.gz
* adodb-5.20.19.tar.gz
* base-1.4.5.tar.gz


* **Install MySQL**

```bash
sudo apt install mysql-server libmysqlclient-dev mysql-client

# set MySQL Security
sudo mysql_secure_installation
```

注：MySQL的安全性请根据个人需求设置。

* **Install PHP**

```bash
sudo apt install apache2

sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php 
sudo apt update
sudo apt install libapache2-mod-php5.6 php5.6 php5.6-common php5.6-gd php5.6-cli php5.6-xml php5.6-mysql
sudo apt install php-pear libphp-adodb

# restart Apache 
sudo systemctl restart apache2
```

* **测试PHP**

在`/var/www/html`中创建一个名为`info.php`的新文件`info.php`，内容如下:

```bash
<?php
phpinfo();
?>
```

使用浏览器，打开：http://localhost/info.php。 类似下图即表明安装好。

![](https://ipichub.oss-cn-hangzhou.aliyuncs.com/2020-07-18-133012.jpg)


* **Install barnyard2**

Barnyard2项目：[Barnyard2](https://github.com/firnsy/barnyard2)

```bash
tar zxvf barnyard2-2-1.13.tar.gz
cd barnyard2-2-1.13
autoreconf -fvi -I ./
./configure --with-mysql --with-mysql-libraries=/usr/lib/x86_64-linux-gnu
sudo make & make install
```

* **测试barnyard2**

```
$ barnyard2 -V

  ______   -*> Barnyard2 <*-
 / ,,_  \  Version 2.1.13 (Build 327)
 |o"  )~|  By Ian Firns (SecurixLive): http://www.securixlive.com/
 + '''' +  (C) Copyright 2008-2013 Ian Firns <firnsy@securixlive.com>
```

* **设置barnyard2配置文件**

```bash
sudo cp barnyard2-2-1.13/etc/barnyard2.conf /etc/snort/
sudo mkdir /var/log/barnyard2
sudo chown snort:snort /var/log/barnyard2
sudo touch /var/log/snort/barnyard2.waldo
sudo chown snort:snort /var/log/snort/barnyard2.waldo
```

* **设置MySQL数据库**

```bash
mysql -u root -p
mysql> create database snort;
mysql> use snort;
mysql> source /opt/barnyard2-2-1.13/schemas/create_mysql;
mysql> CREATE USER 'snort'@'localhost' IDENTIFIED BY '123456';
mysql> grant create, insert, select, delete, update on snort.* to 'snort'@'localhost';
mysql> exit;
```

注意：建议不要使用`123456`作为密码，设置更复杂的密码。

* **设置barnyard2访问数据库用户名等**

```bash
sudo vim /etc/snort/barnyard2.conf

# 文件尾部
output database: log, mysql, user=snort password=123456 dbname=snort host=localhost sensor name=sensor01

# 修改文件权限
sudo chmod 644 /etc/snort/barnyard2.conf
```

* **Install adodb**

```bash
tar zxvf adodb-5.20.14.tar.gz 
sudo mv adodb5 /var/www/html/adodb
```

* **Install BASE**

```bash
tar zxvf base-1.4.5.tar.gz
sudo mv base-1.4.5 /var/www/html/base

# restart apache2
sudo /etc/init.d/apache2 restart
```

* **配置PHP**

```bash
sudo vim /etc/php/5.6/apache2/php.ini 

# add
error_reporting = E_ALL & ~E_NOTICE

# restart Apache2
sudo /etc/init.d/apache2 restart

# 设置目录权限
chown -R root:root /var/www/html
chmod 755 /var/www/html/adodb
```

使用浏览器打开：` http://localhost/base/setup/index.php`。如下图：

![](https://ipichub.oss-cn-hangzhou.aliyuncs.com/2020-07-18-133027.jpg)

参考：[Linux 上搭建 Snort+BASE 入侵检测系统](https://www.cnblogs.com/timdyh/p/12828276.html) 设置即可。

```bash
第 1 步：选择语言 simplified_chinese ，填写 ADOdb 所在目录 /var/www/html/adodb 。

第 2 步：填入数据库的信息，按之前配置的信息填即可（Archive 数据库的信息可以不填）。

第 3 步：填入管理账号：snort，密码：123456。

第 4 步：创建数据表。

第 5 步：提示将显示的信息复制到 /var/www/html/base/base_conf.php 中。
```

设置完成后，使用浏览器打开：`http://localhost/base/base_main.php`。显示如下：

![](https://ipichub.oss-cn-hangzhou.aliyuncs.com/2020-07-18-133030.jpg)

注意：到此，可以说已经全部安装完成。

## 0x03 Run Snort and Barnyard2

*然后，将Snort和barnyard2全部运行起来。*

以`NIDS`模式启动Snort：

```bash
sudo snort -q -u snort -g snort -dev -l /var/log/snort -h 192.168.1.0/16 -c /etc/snort/snort.conf -i eth0 -D
```

以连续模式启动barnyard2：

```bash
sudo barnyard2 -c /etc/snort/barnyard2.conf -d /var/log/snort/ -f snort.log -w /var/log/snort/barnyard2.waldo -g snort -u snort
```

## 0x04 Log explain

*最后，说明一下Snort告警的意义*

在页面查看告警信息，部分内容如下：

```

|   ID   |   特征   |   时间戳   |   来源地址   |   目标地址   |   第4层协议   |
 #0-(2-7350) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:16:11 | 35.224.170.84:80 | 192.168.1.4:51506 | TCP 
 #1-(2-7349) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:16:10 | 35.224.170.84:80 | 192.168.1.4:51506 | TCP 
 #2-(2-7348) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:16:10 | 35.224.170.84:80 | 192.168.1.4:51506 | TCP 
 #3-(2-7347) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:16:10 | 35.224.170.84:80 | 192.168.1.4:51506 | TCP 
 #4-(2-7346) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:16:10 | 35.224.170.84:80 | 192.168.1.4:51506 | TCP 
 #5-(2-7345) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:11:22 | 34.122.121.32:80 | 192.168.1.4:49102 | TCP 
 #6-(2-7344) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:11:21 | 34.122.121.32:80 | 192.168.1.4:49102 | TCP 
 #7-(2-7343) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:11:21 | 34.122.121.32:80 | 192.168.1.4:49102 | TCP 
 #8-(2-7342) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:11:21 | 34.122.121.32:80 | 192.168.1.4:49102 | TCP 
 #9-(2-7341) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:11:21 | 34.122.121.32:80 | 192.168.1.4:49102 | TCP 
 #10-(2-7340) | [snort] stream5: TCP Small Segment Threshold Exceeded | 2021-01-22 11:11:11 | 192.168.1.45:58497 | 192.168.1.4:22 | TCP
 #11-(2-7339) | [snort] stream5: TCP Small Segment Threshold Exceeded | 2021-01-22 11:11:07 | 192.168.1.45:58497 | 192.168.1.4:22 | TCP
 #12-(2-7338) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:10:31 | 91.189.92.20:443 | 192.168.1.4:38138 | TCP
 #13-(2-7337) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:09:52 | 91.189.92.20:443 | 192.168.1.4:38138 | TCP
 #14-(2-7336) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:09:46 | 91.189.92.20:443 | 192.168.1.4:38138 | TCP
 #15-(2-7335) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:09:44 | 91.189.92.20:443 | 192.168.1.4:38138 | TCP
 #16-(2-7334) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:09:42 | 91.189.92.20:443 | 192.168.1.4:38138 | TCP
 #17-(2-7333) | [snort] http_inspect: NO CONTENT-LENGTH OR TRANSFER-ENCODING IN HTTP RESPONSE | 2021-01-22 11:09:42 | 91.189.92.20:443 | 192.168.1.4:38138 | TCP
 #18-(2-7332) | [snort] stream5: TCP session without 3-way handshake | 2021-01-22 11:09:42 | 91.189.92.20:443 | 192.168.1.4:38138 | TCP
```

上面的告警中有三种，如下：

* TCP session without 3-way handshake：TCP连接（三次握手过程）不完整的TCP会话
* TCP Small Segment Threshold Exceeded：TCP中的Segment超过了阈值。
* http_inspect: NO CONTENT-LENGTH OR TRANSFER-ENCODING IN HTTP RESPONSE：在HTTP的Response中，没有内容长度或传输编码。

注：从上面的告警信息，只能说明这些网络链接是异常的，并不能直接判定为网络攻击。


## 0x05 Conclusion

Snort的安装配置比较多，不复杂，需要注意的是，一定要将各种依赖包安装好。此外，从BASE的Web页面来看，Snort的告警信息，并不可以直接判定为网络攻击。对于Snort产生的告警信息，需要进一步的分析。待后续研究。

## Reference

1. [Snort](https://www.snort.org/)
2. [Snort Document: The Snort Project](http://manual-snort-org.s3-website-us-east-1.amazonaws.com/)
3. [在Windows环境下搭建Snort+BASE入侵检测系统](https://www.cnblogs.com/guarderming/p/10282950.html)
4. [WINDOWS下安装Snort](https://blog.csdn.net/jack237/article/details/6900766)
5. [Snort VS Suricata](https://zhuanlan.zhihu.com/p/34329072)
6. [使用Suricata和ELK进行流量检测](https://zhuanlan.zhihu.com/p/64742715)
7. [Suricata规则介绍、以及使用suricata-update做规则管理](https://zhuanlan.zhihu.com/p/36340468)
8. [Security Onion介绍](https://zhuanlan.zhihu.com/p/34072611)
9. [snort的安装、配置和使用](https://blog.csdn.net/qq_37865996/article/details/85088090)
10. [手动打造Snort+barnyard2+BASE可视化报警平台](https://blog.51cto.com/chenguang/2433563)
11. [Linux 上搭建 Snort+BASE 入侵检测系统](https://www.cnblogs.com/timdyh/p/12828276.html)
12. [The LuaJIT Project](http://luajit.org/download.html)