---
layout:     post
title:      Scapy、dpkt数据包文件处理
category: blog
description: 使用Python处理离线数据包文件，常用的两个框架是：Scapy和dpkt。本文说明两者的简单用法，以及两者之间部分不同。
---

## 0x00 缘起

工作经常需要在pcap截包文件中抽取信息，例如抽取pcap文件中dns请求、响应信息等，因此学会了使用python来应对此类需求。在使用的过程中，也踩了很多雷。

首先遭遇的就是内存耗尽的问题，例如Scapy读取pcap文件的接口：`rdpcap()`。此接口在读取pcap文件时，是一次性读完，全部加载进内存，因此在遇到较大的pcap文件时，就可能出现内存耗尽的尴尬了。

其次，在使用`PcapReader()`接口解决内存耗尽问题后，处理数量很多的pcap文件时，就遭遇到耗时太久的问题。

最后，使用`dpkt`库解决耗时长、内存耗尽的问题后，又遇到`dpkt`解析pcap文件中的数据包时，出现各种异常问题。

## 0x01 Scapy处理pcap文件

Scapy安装（设备中使用的是python3）：

```
pip3 install scapy
```

简单的解析数据包，获取TCP数据包中的源IP、目地IP、TCP层的源端口和目的端口，代码如下：

```
#read pcap file, return packet list
pcap_pkts = rdpcap(pcap_file_path)
#traverse packet list
for packet in pcap_pkts:
    if packet.haslayer(IP):
        src_ip = packet[IP].src
        dst_ip = packet[IP].dst
        if packet.haslayer(TCP):
            src_port= packet[TCP].sport
            dst_port= packet[TCP].dport
```

注意：上述代码的最大缺点是在处理较大的pcap文件时，例如1GB的文件，由于`rdpcap()`返回的是pcap文件中的所有数据包，因此将会出现耗尽内存的情况。**此缺点对拥有大内存的朋友不适用。**


## 0x02 Scapy处理大pcap文件

由于需要处理的pcap文件中有大文件，超过1GB，因此很快发现，个人计算机处理此类文件时，内存飙升的尴尬。本着“我不是第一个掉坑的”的原则，一番google后，发现了`PcapReader()`接口。大牛的文章链接为：[python3使用scapy分析修改pcap大文件（1G）](https://blog.csdn.net/qq_38231051/article/details/81460427)

上述的代码就可以修改如下：

```
#read pcap file
with PcapReader(pcap_file_path) as pcap_reader:
    for packet in pcap_reader:
        if packet.haslayer(IP):
            src_ip = packet[IP].src
            dst_ip = packet[IP].dst
            if packet.haslayer(TCP):
                src_port= packet[TCP].sport
                dst_port= packet[TCP].dport
```

这样，内存不大的小本，又可以正确的处理pcap大文件了！

注意：拥有大内存的朋友，请自行忽略这里的改动。

## 0x03 dpkt处理大pcap文件

由于开发了上述数据包处理程序，因此，领导们在面临数据包处理的需求时，毫不犹豫的想到了我。因此，处理的数据包越来越多，pcap文件也越来越大，甚至以TB来计算。此时，尴尬的发现：上述数据包处理代码在处理TB级的pcap文件，小本本一天一夜也没有运行完成。此处郁闷3分钟。

本着“我不是第一个掉坑的”原则，一番google后，发现了`dpkt`库。大牛的文章链接：[dpkt和scapy对处理大pcap包分析对比](https://blog.csdn.net/qq_38231051/article/details/82019782)。

dpkt安装：

```
pip3 install dpkt
```

于是上面的代码修改如下：

```
def inet_to_str(inet):
    try:
        return socket.inet_ntop(socket.AF_INET, inet)
    except:
        return False

f = open(pcap_file_path, 'rb')
pcap = dpkt.pcap.Reader(f)
for ts,buf in pcap:
    eth_packet = dpkt.ethernet.Ethernet(buf)
    if eth.type != dpkt.ethernet.ETH_TYPE_IP: continue
    
    ip_layer = eth_packet.data
    src_ip = inet_to_str(ip_layer.src)
    dst_ip = inet_to_str(ip_layer.dst)
    ip_proto = ip_layer.p
    
    if ip_proto == dpkt.ip.IP_PROTO_TCP:
        tcp_layer = ip_layer.data
        src_port = tcp_layer.sport
        dst_port = tcp_layer.dport
```

这样，小本本又可以快速的处理pcap大文件了！！

注意：`dpkt`的文档极其缺乏，本人是边翻源码，边开发上述代码的。对性能要求不高的话，完全没必要使用这个库。当然，使用这个库的好处是，在小本上Scapy处理1.9GB文件需要4000s左右，使用dpkt处理时，仅需80S左右。这个性能的提升，领导非常满意。

## 0x04 小结

猜测：Scapy读取pcap文件时，对每一个数据包进行按层解析，得到解析好的数据包。这样，开发者只需要使用解析好的数据包对象的属性，以及接口，就可以完成常见的工作。而dpkt在读取数据包时，并没有先解析好，而是在使用时解析。（理由：pcap文件中有非IP数据包，在没有做这个处理时，处理程序会异常退出。）

综上：在处理小pcap文件时，Scapy是个不错的选择，可以避免解析数据包的烦恼。而在要求高性能的处理pcap文件时，dpkt比Scapy要更好，缺点是看源码吧，没有啥文档可看。

## 参考

1. [dpkt和scapy对处理大pcap包分析对比](https://blog.csdn.net/qq_38231051/article/details/82019782)
2. [python3使用scapy分析修改pcap大文件（1G）](https://blog.csdn.net/qq_38231051/article/details/81460427)
3. [github: dpkt](https://github.com/kbandla/dpkt)
4. [python dpkt 解析 pcap 文件](http://www.cnblogs.com/bonelee/p/10375742.html)
5. [Scapy](https://scapy.net/)