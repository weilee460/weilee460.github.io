---
layout:     post
title:      OpenVPN Service Build
category: blog
description: OpenVPN是常见的科学上网工具之一，可以保护用户的上网数据。在这个隐私受到越来越多破坏的环境中，一个隐私保护工具是不可缺少的。本文简述OpenVPN服务的搭建，记录搭建服务的过程以及遇到的问题。
---

## 0x00 Origin

在天朝，为了少受“百毒”的毒害，需要学会科学上网，使用`google`。`OpenVPN`是常见的科学上网的工具之一，因此尝试构建个人的`OpenVPN`服务。

## 0x01 Install

配置环境server：Ubuntu 18.04LTS，client：Windows10。

安装编译依赖软件包：

```
sudo apt install lzop liblzo2-2 liblzo2-dev libpam0g libpam0g-dev
```

OpenVPN编译安装：

```
./configure && make && make-install
```

## 0x02 Static Configuration

### 0x0200 OpenVPN Config

生成static key，文件名为static.key：

```
openvpn --genkey --secret static.key
```

OpenVPN Server config file，来源于OpenVPN官网：

```
#static-server.conf
dev tun
ifconfig 10.8.0.1 10.8.0.2
secret static.key
```

OpenVPN Client config file，来源于OpenVPN官网:

```
#client.ovpn
remote myremote.mydomain
dev tun
ifconfig 10.8.0.2 10.8.0.1
secret static.key
```

### 0x0201 System config

Ubuntu 18.04LTS开启内核转发，配置文件为`/etc/sysctl.conf`.注意：设置完成需要重启系统.

```
# IPv4 forward，将此配置项的值修改为1
net.ipv4.ip_forward=1

# IPv6 forward
net.ipv6.conf.all.forwarding=1
```


设置系统路由表，重启失效：

```
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j SNAT --to-source server公网地址
```

### 0x0202 service start

启动OpenVPN服务：

```
sudo openvpn --config ~/openvpn-config/static-server.conf
```

### 0x0203 Advantage and Disadvantage

OpenVPN官网描述如下：

> Static Key advantages:
>> * Simple Setup
>> * No X509 PKI (Public Key Infrastructure) to maintain
>
> Static Key disadvantages:
>> * Limited scalability — one client, one server
>> * Lack of perfect forward secrecy — key compromise results in total disclosure of previous sessions
>> * Secret key must exist in plaintext form on each VPN peer
>> * Secret key must be exchanged using a pre-existing secure channel

总结起来，就是static key方式仅仅支持一对client-server。优点是可以减少`GFW`的干扰。

## 0x03 TLS Configuration

### 0x0300 Cert Generate

此处使用OpenVPN官方推荐的easy-rsa工具，生成证书等。

证书生成参考[完整CentOS搭建OpenVPN服务环境图文教程](http://ju.outofmemory.cn/entry/71676)即可。

client端文件有：

```
ca.crt
myself_client.crt
myself_client.key
```

server端文件有：

```
ca.crt
myself_server.key
myself_server.crt
dh.pem
```

### 0x0301 config file

OpenVPN Server config file，供参考：

```
#tls-server.conf
local xxx.xxx.xxx.xxx（Server IP）
port 1194
#可选择UDP、TCP
proto udp
dev tun
#需修改为自定义的路径
ca /etc/openvpn/ca.crt
cert /etc/openvpn/myself-server.crt
key /etc/openvpn/myself-server.key
dh /etc/openvpn/dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.4.4"
keepalive 10 120
max-clients 100
persist-key
persist-tun
status openvpn-status.log
verb 3
```

OpenVPN Client config file，供参考：

```
#tls-client.ovpn
client
dev tun
#与server相同
proto udp
#主要这里修改成Server ip and port
remote xxx.xxx.xxx.xx port
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt //这里需要证书
cert qingliu.crt
key qingliu.key
verb 3
```

### 0x0303 service start

```
#配置文件需修改为自定义的
sudo openvpn --config ~/openvpn-server/tls-server.conf
```

### 0x0304 配置文件说明

OpenVPN config file的配置项说明：

1. `secret` --- 指定static key文件；
2. `port` --- 指定openvpn的通信端口；默认端口是`1194`；
3. `dev` --- 指定openvpn通信方式；有：`tun`和`tap`两种；
4. `proto` --- 指定openvpn服务的通信协议；有：`tcp-server`和`tcp-client`；默认是`udp`，无需指定；使用证书认证方式时，可以配置为`TCP` or `UDP`，指定OpenVPN在传输层使用的协议；
5. `push` --- 指定openvpn服务使用的DNS；例如：` push "dhcp-option DNS 208.67.222.222" `；
6. `compress` --- 指定openvpn服务的通信使用的压缩方式；例如：`lz4-v2`；对于老旧版本的，可以使用`comp-lzo`， 从而兼容过时的压缩方式；
7. `ca` --- 使用证书认证时，ca证书的路径；
8. `cert` --- 使用证书认证时，client or server 端的证书路径；
9. `key` --- 使用证书认证时，client or server 端的证书key（私钥）路径；
10. `dh` --- 使用证书认证时，server端的密钥dh协商参数文件；
11. `server` --- server端指定client得到的子网IP以及子网掩码；
12. `tls-auth` --- 启用`tls auth`功能；注意，此配置项后需要紧跟`key`文件，文件后的值在`server`和`client`是不同的，`client`端的值是`1`，`server`端是`0`；
13. `auth` --- 与`tls-auth`配合使用，指定计算`HMAC`的计算方法；可选的方法有`MD5`，`SHA1`，`SHA256`，`SHA512`，对应的`HMAC`长度分别是`16`字节、`20`字节、`32`字节、`64`字节；

注意：OpenVPN提供的有配置样例文件，路径为：`openvpn-2.3.4/sample/sample-config-files`。参考其中的配置文件，进行配置即可。

## 0x04 Troubleshooting

### 0x0400 No TAP Error

在Windows的客户端，添加配置文件以及证书、key等文件后，使用OpenVPN客户端连接时，显示`There are no TAP --- Windows adapters on this system`的错误信息，使用错误信息，搜索后，参考此链接上的解决办法[openvpn There are no TAP-Windows adapters on this system](https://blog.csdn.net/palmer_kai/article/details/93738198)，重新安装OpenVPN的客户端后，解决问题。

### 0x0401 连接成功，网页无法打开

在使用OpenVPN客户端连接服务器成功后，使用浏览器打开网时，网页打不开。猜测是服务端的路由问题，毕竟系统的内核转发已经打开，因此给Ubuntu添加路由后，解决此问题。

### 0x0402 UDP数据包连续问题

**问题描述：**

在使用`UDP`作为层4的协议时，client断开和server端的连接后，server端仍然会给client端发送`OpenVPN`的`P_DATA_V2`类型的报文，并且会持续一段时间。

**问题猜测：**

由于`UDP`是无连接的协议，因此client端`disconnect`之后，server端不能像使用`TCP`那样，收到`FIN/RST`的报文，因此无法判断client是否还在线。对于client端访问的网站等，其网站数据还会通过server端传回给client端，并进一步触发client回送`ICMP`报文，告诉server端：客户端是`Port unreachable`。对于`OpenVPN`客户端，应该会有某种通知机制，告诉server端，它下线了；从而提高服务端的性能（都是钱呀 :-) :-) ）。进一步，在`OpenVPN`的软件包的`sample`目录内的`server.conf`文件中，发现了`explicit-exit-notify`的配置项。但`client.conf`文件中没有此配置项。那么是否可以使用到client的配置中呢？

**解决办法：**

在客户端的配置文件中，添加`explicit-exit-notify 1`的配置项。经过测试，解决此问题。

**说明：**

1. 此配置项只有在选择`UDP`时，才能生效。因为`TCP`是有连接的，无需此机制。

## Reference

1. [OpenVPN Windows client download](https://swupdate.openvpn.org/community/releases/openvpn-install-2.4.4-I601.exe)
2. [Static Key Mini-HOWTO](https://openvpn.net/community-resources/static-key-mini-howto/)
3. [完整CentOS搭建OpenVPN服务环境图文教程](http://ju.outofmemory.cn/entry/71676)
4.  [easy-rsa](https://github.com/OpenVPN/easy-rsa)