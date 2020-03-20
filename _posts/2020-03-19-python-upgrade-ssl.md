---
layout:     post
title:     Python SSL Module not Found
category: blog
description: Python编译升级，在使用pip install库时，出现ssl module未找到的错误。查了一下，此错误是很普遍的，因此这里做一个记录。
---

## 0x00 Introduction

由于测试需要将Python升级到Python3.6的版本，考虑到该服务器还提供其他的服务，因此使用源码编译方式升级Python。下载Python3源码包后，编译安装`./configure & make & make install`。在安装第三方库时，发现SSL module的报错问题。经过搜索，发现好多blog提供的解决方案，并不能解决该问题。因此本文做一个记录。

## 0x01 Trouble

Server OS: Ubuntu 14.04 LTS

Python Version: V3.6.8

在执行`pip3 install numpy`时，报错如下：

```bash
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Collecting pip
  Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/pip/

......

  Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)) - skipping
  
......

No matching distribution found for pip
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.

```

从报错信息来看，是SSL module没有找到。因此返回查看Python编译信息。注意到其中的一段告警信息如下：

```bash
Failed to build these modules:
_hashlib              _ssl
```

该告警信息清晰的说明了**ssl module编译失败**。（没看编译告警信息的错误，汗!!）

根据Python的依赖关系说明，还需要升级openssl。因此编译安装openssl-1.1.0j。升级Openssl之后，再次编译Python。查看其中的编译告警信息，依然发现ssl模块编译错误的问题（尝试`./configure --enable-optimizations`，问题依然。）。好奇怪！！

在参考【1】的blog后，忽然想到个人的这个环境也应该是这种问题。即，链接库的位置问题。

查看系统中的libssl.so：`find / -name libssl.so`

发现结果中有：

```
/lib/x86_64-linux-gnu/libssl.so.1.0.0
```

问题很明确了：**安装openssl是1.1.0的，编译得到的是libssl.so.1.1；而这里依然是libssl.so.1.0.0。**

再进一步：

>系统是先根据`/etc/ld.so.conf`中指定的路径，搜索链接库；搜索不到时，再搜索`/usr/lib`等路径下的链接库。

也就是说，Python编译时，先链接了`/lib/x86_64-linux-gnu/libssl.so.1.0.0`，不在链接`/usr/lib/libssl.so.1.1`。由于以来版本问题，因此编译时ssl模块编译失败。

修改系统中的链接库：

```bash
sudo cp libssl.so.1.1 /lib/x86_64-linux-gnu
sudo cp libssl.a /lib/x86_64-linux-gnu
# sudo rm /lib/x86_64-linux-gnu/libssl.so.1.0.0

sudo cp libcrypto.so.1.1 /lib/x86_64-linux-gnu
sudo cp libcrypto.a /lib/x86_64-linux-gnu
# sudo rm /lib/x86_64-linux-gnu/libcrypto.so.1.0.0

sudo cp libssl.a /usr/lib/x86_64-linux-gnu
sudo cp libcrypto.a /usr/lib/x86_64-linux-gnu

sudo rm /usr/lib/x86_64-linux-gnu/libcrypto.so
sudo rm /usr/lib/x86_64-linux-gnu/libssl.so

sudo ln -s /lib/x86_64-linux-gnu/libcrypto.so.1.1 /usr/lib/x86_64-linux-gnu/libcrypto.so
sudo ln -s /lib/x86_64-linux-gnu/libssl.so.1.1 /usr/lib/x86_64-linux-gnu/libssl.so
```

修改后，再次编译Python即可。成功解决 :-). :-)

注意：**在后续的使用中发现openssh依赖于libssl.so.1.0.0，删除将会导致ssh不可用**。因此不应该删除。上述已注释掉删除操作。

## Reference

1. [ \_openssl.so: undefined symbol: OPENSSL_sk_num](https://blog.csdn.net/jisuanji2121/article/details/53944648)
