---
layout:     post
title:     Hadoop Cluster Configuration
category: blog
description: 网上常见单机以及伪分布式环境搭建，本文描述Hadoop Cluster完全分布式环境搭建。
---

## 0x00 Introduction

当前由于需要处理的数据量越来越大，单机存储、分析等无法应对，因此考虑升级到Hadoop生态圈技术来应对。对于此类技术，首先需要配置一个测试环境，来测试原有的存储、分析等功能。因此本文记录一下全分布式环境的配置。

## 0x01 Environment Explaination

**Base Environment**

* OS:  Ubuntu 18.04LTS(X64)
* PC Count: 3
* Master Node: 1
* Slave Node: 2

**Distribution Component Version**

* Java: 1.8.0_191
* Hadoop: 2.7.7
* Hive: 3.1.2
* Spark: 2.4.4

## 0x02 Environment Configuration

1. 首先设置主机名称：

```bash
#分别修改主机名称
hostnamectl set-hostname hadoop-master

hostnamectl set-hostname hadoop-slave1

hostnamectl set-hostname hadoop-slave2
```

2. 设置节点名称和IP对应关系；

```bash
vim /etc/hosts
# add flow
192.168.1.133   hadoop-master
192.168.1.142   hadoop-slave1
192.168.1.127   hadoop-slave2
```

注意：需要删除节点名称对应的其他IP地址。例如：

```bash
127.0.1.1 hadoop-master
```

类似的上述名称和IP映射关系，将导致Hadoop集群找不到其他的节点。例如两个DataNode都找不到。**此处是大坑，好尴尬，浪费了一天。**

3. 设置ssh免密登陆；

```bash
# generate ssh-key
$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/dev/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/dev/.ssh/id_rsa.
Your public key has been saved in /home/dev/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx dev@dev
The key's randomart image is:
+---[RSA 2048]----+
|  +              |
| = o   =         |
|o + E + =        |
|o+o* = o =       |
|O+o.= = S o      |
|+Bo. . = = +     |
|+o+.  o o = .    |
|o+.    . .       |
|o                |
+----[SHA256]-----+

#在hadoop-master节点上使用ssh-copy-id，创建免密登陆
ssh-copy-id dev@hadoop-slave1
ssh-copy-id dev@hadoop-slave2
```

4. Config Java Environment Variable；

```bash
vim /etc/profile

# add environment variable, need modify to yourself path

export JAVA_HOME="/home/dev/jdk1.8.0_191"
export PATH="$JAVA_HOME/bin:$PATH"
export CLASSPATH=".:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar"
```

Verify Java Configuration：

```bash
$ java -version

java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```

## 0x03 Hadoop Cluster Configuration

Hadoop version info: Hadoop-2.7.7

Hadoop Cluster有三种角色的节点：NameNode，Secondary NameNode和DataNode。此Cluster中分布如下：NameNode和Secondary NameNode部署在同一个pc上，其余两个Slave节点均作为DataNode。

### 0x0301 HDFS Cluster Configuration

1. get hadoop from Apache Hadoop WebSite，Set Hadoop environment variable；

```bash
export HADOOP_HOME="/home/dev/hadoop-2.7.7"
export PATH="$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH"

```

2. Verify Hadoop Environment setting：

```bash
$ hadoop version

Hadoop 2.7.7
Subversion Unknown -r c1aad84bd27cd79c3d1a7dd58202a8c3ee1ed3ac
Compiled by stevel on 2018-07-18T22:47Z
Compiled with protoc 2.5.0
From source with checksum 792e15d20b12c74bd6f19a1fb886490
This command was run using /home/dev/hadoop-2.7.7/share/hadoop/common/hadoop-common-2.7.7.jar
```

3. Hadoop Cluster配置

Hadoop Cluster Node 有两类：Master和Slave。另一种按照角色分类是有三类，分别是：NameNode，Secondary NameNode和DataNode。此处的节点角色配置是将NameNode和Secondary NameNode部署在同一个物理设备上，DataNode分别位于两个Slave节点上。代价是存在单点故障问题，好处是无需准备另一台物理设备。**最重要的是：没钱 :-) :-)**

Master节点和Slave节点配置相同的配置文件有两个：`hadoop-env.sh`和`core-site.xml`。角色不同，配置不同的配置内容的配置文件是`hdfs-site.xml`。注意配置项不同，配置项的value也不同。

MasterNode and SlaveNode same configuration：

1. `etc/hadoop/hadoop-env.sh`文件配置如下：

```bash
    #set JAVA_HOME environment variable
    vim hadoop-2.7.7/etc/hadoop/hadoop-env.sh

    #Add export to file end
    export JAVA_HOME="/home/dev/jdk1.8.0_191"
```

2. `etc/hadoop/core-site.xml`文件配置如下：

```xml
    <configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://hadoop-master:9000</value>
        </property>
        <property>
                <name>io.file.buffer.size</name>
                <value>131072</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/dev/hadoop_tmp</value>
        </property>
    </configuration>
```

**MasterNode and SlaveNode differrent configuration：**

1. `etc/hadoop/hdfs-site.xml` NameNode/MasterNode 配置：

```xml
<configuration>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>

        <property>
                <name>dfs.namenode.name.dir</name>
                <value>/home/dev/hadoop_name</value>
        </property>

        <property>
                <name>dfs.blocksize</name>
                <value>268435456</value>
        </property>

        <property>
                <name>dfs.namenode.handler.count</name>
                <value>100</value>
        </property>
        <property>
            <name>dfs.namenode.secondary.http-address</name>
            <value>hadoop-master:50090</value>
        </property>
        <property>
                <name>dfs.permissions</name>
                <value>false</value>
        </property>
</configuration>
```

2. `etc/hadoop/hdfs-site.xml` DataNode/SlaveNode 配置：

```xml
<configuration>
        <property>
                <name>dfs.replication</name>
                <value>2</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>/home/dev/hadoop_data</value>
        </property>
        <property>
                <name>dfs.namenode.secondary.http-address</name>
                <value>hadoop-master:50090</value>
        </property>
</configuration>
```

### 0x0302 YARN Configuration

YARN Cluster中，有两类角色的节点：ResourceManager和NodeManager。

ResourceManager和NodeManager节点相同的配置是`etc/hadoop/yarn-env.sh`，其配置如下：

```bash
# Set JAVA_HOME environment variable
export JAVA_HOME="/home/dev/jdk1.8.0_191"
```

ResourceManager角色的`etc/hadoop/yarn-site.xml` 配置：

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.acl.enable</name>
                <value>false</value>
        </property>
        <property>
                <name>yarn.admin.acl</name>
                <value>*</value>
        </property>
        <property>
                <name>yarn.log-aggregation-enable</name>
                <value>false</value>
        </property>
        <property>
                <name>yarn.nodemanager.vmem-check-enabled</name>
                <value>false</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>hadoop-master:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>hadoop-master:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>hadoop-master:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>hadoop-master:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>hadoop-master:8089</value>
        </property>
</configuration>
```

`etc/hadoop/yarn-site.xml` NodeManager 文件配置：

```xml
<configuration>

<!-- Site specific YARN configuration properties -->
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.acl.enable</name>
                <value>false</value>
        </property>
        <property>
                <name>yarn.admin.acl</name>
                <value>*</value>
        </property>
        <property>
                <name>yarn.log-aggregation-enable</name>
                <value>false</value>
        </property>
        <!-- add resourcemanager info in nodemanager -->
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>hadoop-master:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.scheduler.address</name>
                <value>hadoop-master:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>hadoop-master:8031</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>hadoop-master:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>hadoop-master:8089</value>
        </property>
</configuration>
```

注：2020-06-01，修正在nodemanager节点上遗漏resourcemanager节点信息的错误。缺少这个信息，nodemanager启动一段时间后，会因为无法连接resourcemanager的错误，而异常退出。

### 0x0303 MapReduce Configuration

`etc/hadoop/mapred-site.xml`文件配置：

```bash
#利用模版文件，创建配置文件
cp mapred-site.xml.template mapred-site.xml
```

```xml
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.address</name>
                <value>hadoop-master:10020</value>
        </property>
        <property>
                <name>mapreduce.jobhistory.webapp.address</name>
                <value>hadoop-master:19888</value>
        </property>
</configuration>
```

启动Hadoop Cluster并验证HDFS:

```bash
# MasterNode hadoop-2.7.7/sbin
$ ./start-dfs.sh

# Verification hadoop
$ jps
9802 SecondaryNameNode
9450 NameNode
9979 Jps

# DataNode 结果如下
$ jps
18193 DataNode
18294 Jps
```

Web页面验证，打开浏览器，输入：`http://192.168.1.133:50070`。注意节点数目，存储容量等。显示正常的话，即可说明hdfs配置正确。

验证HDFS后，保持HDFS启动状态，验证Yarn:

```bash
# MasterNode hadoop-2.7.7/sbin
$ ./start-yarn.sh

# Verification Yarn on MasterNode
$ jps
11600 SecondaryNameNode
11769 ResourceManager
12031 Jps
11247 NameNode

# Verification Yarn on DataNode
$ jps
37124 Jps
36489 NodeManager
36287 DataNode
```

打开浏览器，输入：`http://192.168.1.133:8088`。注意节点信息等。显示正常的话，即可说明Yarn配置正确。

### 0x0304 HDFS Client Access

我们经常需要在本地的开发设备上，读写Hadoop Cluster中的文件，因此需要部署Hadoop的http的访问服务。

hadoop-master node中的`hadpp-2.7.7/etc/hadoop/hdfs-site.xml`配置文件增加如下几行：

```xml
<property>
    <name>dfs.http.address</name>
    <value>0.0.0.0:50070</value>
</property>

<property>
    <name>dfs.datanode.http.address</name>
    <value>hadoop-slave1:50075</value>
</property>

<property>
    <name>dfs.datanode.http.address</name>
    <value>hadoop-slave2:50075</value>
</property>

<property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
</property>
```

hadoop-slave1 node中的`hadpp-2.7.7/etc/hadoop/hdfs-site.xml`配置文件增加如下几行：

```xml
<property>
    <name>dfs.http.address</name>
    <value>hadoop-master:50070</value>
</property>

<property>
    <name>dfs.datanode.http.address</name>
    <value>0.0.0.0:50075</value>
</property>

<property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
</property>
```

hadoop-slave2 node中的`hadpp-2.7.7/etc/hadoop/hdfs-site.xml`配置文件增加如下几行：

```xml
<property>
    <name>dfs.http.address</name>
    <value>hadoop-master:50070</value>
</property>

<property>
    <name>dfs.datanode.http.address</name>
    <value>0.0.0.0:50075</value>
</property>

<property>
    <name>dfs.webhdfs.enabled</name>
    <value>true</value>
</property>
```

修改开发设备本地的hosts文件，添加Cluster节点名称与IP对应关系，替换本地的hosts文件（Mac hosts文件 path:/private/etc/hosts）：

```bash
192.168.1.133   hadoop-master
192.168.1.142   hadoop-slave1
192.168.1.127   hadoop-slave2
```

开发设备安装hdfs库：

```bash
pip3 install hdfs
```

测试python code：

```python
#! /usr/bin/env python3
# -*- coding: utf-8 -*-

import os
import sys

from hdfs.client import Client

def test_read_hdfs():
    lines = []
    file_name = "/test_data/hosts"
    client = Client("http://192.168.1.133:50070")
    with client.read(file_name,encoding='utf-8',delimiter='\n') as reader:
        for line in reader:
            print(line.strip())

    return lines

if __name__=="__main__":
    print("hello python")
    test_read_hdfs()
```

Execute and Output:

```bash
$ python3 test_hdfs.py
hello python
127.0.0.1	localhost

192.168.1.133   hadoop-master
192.168.1.142   hadoop-slave1
192.168.1.127   hadoop-slave2

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Success!!  :-) :-)

### 0x0305 Hadoop Troubleshooting

1. Cluster ID在namenode和datanode中，不一致，导致datanode中dfs无法启动；参考【26】解决此错误。
2. 

## 0x04 Hive Configuraion

### 0x0401 MySQL Configuration

Hive需要使用MySQL存储metadata，因此需要在ResourceManager节点上安装MySQL。此Cluster中，ResourceManager角色部署在Hadoop NameNode上。MySQL安装配置：

1. Install MySQL;

```bash
$ sudo apt install mysql-server

#完成安装后，执行安全配置
$ sudo mysql_secure_installation
```

查看MySQL的运行状态：

```bash
$ systemctl status mysql.service
● mysql.service - MySQL Community Server
   Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2019-12-21 11:14:24 CST; 4h 34min ago
  Process: 1308 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid (code=exited, status=0/SUCCESS)
  Process: 1253 ExecStartPre=/usr/share/mysql/mysql-systemd-start pre (code=exited, status=0/SUCCESS)
 Main PID: 1314 (mysqld)
    Tasks: 35 (limit: 4915)
   CGroup: /system.slice/mysql.service
           └─1314 /usr/sbin/mysqld --daemonize --pid-file=/run/mysqld/mysqld.pid

12月 21 11:14:23 hadoop-master systemd[1]: Starting MySQL Community Server...
12月 21 11:14:24 hadoop-master systemd[1]: Started MySQL Community Server.
```

查看设备MySQL服务端口使用情况：

```bash
netstat -an | grep 3306

tcp6       0      0 127.0.0.1:3306     :::*             LISTEN  
```

注意：上面显示的MySQL服务绑定的IP为本地IP。这样将导致其他的设备无法访问此MySQL服务。需要将`127.0.0.1`本地IP修改为`0.0.0.0`，允许任意的IP访问服务。

修改MySQL服务绑定的IP

```bash
#编辑mysql bind端口
vim /etc/mysql/mysql.conf.d/mysqld.cnf

#原来
bind-address = 127.0.0.1

#修改为
bind-address = 0.0.0.0
```

重启MySQL服务：

```
$ service mysql restart
```

Login MySQL with root user:

```bash
$ sudo mysql -uroot -p
```

创建新用户：

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' IDENTIFIED BY "123456"; 
```

查看MySQL数据库中的用户：

```bash
mysql> use mysql;
Database changed
mysql> select host,user,password from user;
+--------------+------+-------------------------------------------+
| host         | user | password                                  |
+--------------+------+-------------------------------------------+
| localhost    | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
| 192.168.1.1 | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
+--------------+------+-------------------------------------------+
2 rows in set (0.00 sec)
```

修改用户对应的host值，赋予远程登陆的权限：

```bash
mysql> use mysql;
Database changed
mysql> grant all privileges  on *.* to root@'%' identified by "password";
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> select host,user,password from user;
+--------------+------+-------------------------------------------+
| host         | user | password                                  |
+--------------+------+-------------------------------------------+
| localhost    | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
| 192.168.1.1 | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
| %            | root | *A731AEBFB621E354CD41BAF207D884A609E81F5E |
+--------------+------+-------------------------------------------+
3 rows in set (0.00 sec)

```

MySQL数据库用户host修改，允许用户远程连接MySQL：

```SQL
use mysql;

update user set host = '%' where user = 'root';
```

设置用户密码安全等级：

```bash
mysql> set global validate_password_policy=0;

Query OK, 0 rows affected (0.00 sec)
```


### 0x0402 Hive Local Configuration

Hive配置：

编辑`apache-hive-3.1.2-bin/conf/hive-env.sh`文件：

```bash
vim hive-env.sh

export HADOOP_HOME="/home/dev/hadoop-2.7.7"
export HIVE_CONF_DIR="/home/dev/apache-hive-3.1.2-bin/conf"
export HIVE_AUX_JARS_PATH="/home/dev/apache-hive-3.1.2-bin/lib"
```

编辑`apache-hive-3.1.2-bin/conf/hive-site.xml`文件：

```xml
<property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hivedb?createDatabaseIfNotExist=true&amp;useSSL=false</value>
    <description>JDBC connect string for a JDBC metastore</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
    <description>Driver class name for a JDBC metastore</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>feng</value>
    <description>username to use against metastore database</description>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>123456</value>
    <description>password to use against metastore database</description>
  </property>

  <property>
    <name>hive.metastore.warehouse.dir</name>
    <value>hdfs://hadoop-master:9000/hive/warehouse</value>
    <description>hive default warehouse, if nessecory, change it</description>
  </property>

  <property>
          <name>hive.querylog.location</name>
          <value>hdfs://hadoop-master:9000/hive/log</value>
          <description>Location of Hive run time structured log file</description>
  </property>

  <property>
          <name>hive.exec.scratchchdir</name>
          <value>hdfs://hadoop-master:9000/hive/tmp</value>
          <description>HDFS root scratch dir for Hive jobs</description>
  </property>
```

Verify Hive Configuration:

```SQL
#查看Hive中的数据库
hive> show databases;
OK
default
Time taken: 0.717 seconds, Fetched: 1 row(s)

#创建Database
hive> CREATE DATABASE test;
OK
Time taken: 0.307 seconds

#验证数据库创建
hive> show databases;
OK
default
test
Time taken: 0.015 seconds, Fetched: 2 row(s)
```

验证在Slave节点上，Hive功能正常：

```bash
#slave节点进入hive cmd
$ hive
```

查看数据库：

```SQL
hive> show databases;
OK
default
test
Time taken: 0.596 seconds, Fetched: 2 row(s)
```

其他的Slave节点，同样验证即可。到此，即可以证明Hive集群配置正确。


### 0x0403 HiveServer2 Configuration

配置参考：[Setting Up HiveServer2](https://cwiki.apache.org/confluence/display/Hive/Setting+up+HiveServer2)。配置项说明如下：

> * hive.server2.thrift.min.worker.threads – Minimum number of worker threads, default 5.
> * hive.server2.thrift.max.worker.threads – Maximum number of worker threads, default 500.
> * hive.server2.thrift.port – TCP port number to listen on, default 10000.
> * hive.server2.thrift.bind.host – TCP interface to bind to.


`conf/hive-site.xml`文件，增加下面4个配置项：

```xml
<property>
          <name>hive.server2.thrift.min.worker.threads</name>
          <value>5</value>
          <description>hive server2 worker min threads</description>
  </property>

  <property>
          <name>hive.server2.thrift.max.worker.threads</name>
          <value>300</value>
          <description>hive server2 max worker threads</description>
  </property>

  <property>
          <name>hive.server2.thrift.port</name>
          <value>10000</value>
          <description>hive server2 thrift listening port</description>
  </property>

  <property>
          <name>hive.server2.thrift.bind.host</name>
          <value>hadoop-master</value>
          <description>hive server2 thift bind host IP</description>
 </property>
```

Hadoop Cluster配置需要修改，以便允许hive对hdfs文件系统的访问。注意：需要在hadoop cluster所有节点上更新配置。`hadoop-2.7.7/etc/hadoop/core-site.xml`文件，增加下述两项:

```xml
<property>
    <name>hadoop.proxyuser.dev.hosts</name>
    <value>*</value>
</property>

<property>
    <name>hadoop.proxyuser.dev.groups</name>
    <value>*</value>
</property>
```

**注意：** 上述配置不支持beeline工具的连接。浪费了整整一天时间。傻！！！

启动hiveserver2：

```bash
hadoop-master:~/apache-hive-3.1.2-bin/bin$ hiveserver2

#后台启动
hadoop-master:~/apache-hive-3.1.2-bin/bin$ hive --service hiveserver2 &

```

验证hiveserver2 service port:

```bash
netstat -apn |grep 10000
```

使用pyhive第三方库，测试HiveServer2连接，并获取数据：

```python
#! /usr/bin/env python3
# -*- coding: utf-8 -*-

import os
import sys
from pyhive import hive

def create_database_hiveconnect():
    hive_host = '192.168.1.133'
    hive_port = 10000
    hive_username = "test"
    hive_dbname = "test"
    hive_auth = "NOSASL"

    conn = hive.connect(host=hive_host, port=hive_port, \
        username=hive_username, database=hive_dbname,auth=hive_auth)
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM pokes")
    results_list = cursor.fetchall()
    print(results_list)

    return
```

测试上述的HiveServer查询代码：

```bash
$ python3 test_hive.py

hello python
[(10, 'zhangsan')]
```

**注意：** 官方给出的测试代码是使用pyhs2的，这个库早就没有人维护了。不建议使用。这里使用pyhive，Dropbox官方开源并维护的。


## 0x04 Spark Configuration

### 0x0401 Spark Configuration

首先配置`spark-env.sh`文件：

```bash
#create spark-env.sh file
cp spark-env.sh.template spark-env.sh
#edit file
vim spark-env.sh

# add environment variable
export JAVA_HOME=/home/dev/jdk1.8.0_191
export HADOOP_HOME=/home/dev/hadoop-2.7.7
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export SPARK_HOME=/home/dev/spark-2.4.4-bin-with-hadoop
export SPARK_LIBRARY_PATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$HADOOP_HOME/lib/native
export SPARK_MASTER_PORT=7077
export SPARK_MASTER_HOST=hadoop-master
export PYSPARK_PYTHON=/usr/bin/python3
```

再次，配置`slaves`文件：

```bash
#create slaves configuration file
cp slaves.template slaves

#edit slaves file
vim slaves

# delete localhost, then add follow info
hadoop-master
hadoop-slave1
hadoop-slave2
```

配置验证：

测试程序文件如下：

```python
#! /usr/bin/env python3
# -*- coding: utf-8 -*-

import sys
import os
import time

from pyspark.ml.fpm import FPGrowth
from pyspark.ml.fpm import PrefixSpan
from pyspark.sql import Row,SparkSession
from pyspark import SparkContext

if __name__=="__main__":
    spark = SparkSession.builder.appName("PrefixSpanTest")\
        .getOrCreate()

    sc = spark.sparkContext

    df = sc.parallelize([Row(sequence=[[1, 2], [3]]),
                     Row(sequence=[[1], [3, 2], [1, 2]]),
                     Row(sequence=[[1, 2], [5]]),Row(sequence=[[2],[1,3,4]]),
                     Row(sequence=[[6]])]).toDF()

    prefixSpan = PrefixSpan(minSupport=0.5, maxPatternLength=5,
                        maxLocalProjDBSize=32000000)

    # Find frequent sequential patterns.

    ret = prefixSpan.findFrequentSequentialPatterns(df)
    print(type(ret))
    ret.show()
```

执行上述的测试代码文件：

```bash
hadoop-master:~/spark-2.4.4-bin-with-hadoop/bin$ ./spark-submit spark_demo.py
```

Excution Output:

```bash
hadoop-master:~/spark-2.4.4-bin-with-hadoop/bin$ ./spark-submit spark_demo.py
19/12/24 15:21:27 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
19/12/24 15:21:28 INFO spark.SparkContext: Running Spark version 2.4.4

....

NativeMethodAccessorImpl.java:0) finished in 0.060 s
19/12/24 15:21:34 INFO scheduler.DAGScheduler: Job 6 finished: showString at NativeMethodAccessorImpl.java:0, took 0.067907 s
+--------+----+
|sequence|freq|
+--------+----+
|   [[3]]|   3|
|   [[2]]|   4|
|   [[1]]|   4|
|[[1, 2]]|   3|
+--------+----+

.........

19/12/24 15:21:34 INFO util.ShutdownHookManager: Deleting directory /tmp/spark-75d73fbc-087b-4d87-9e10-2c28846a2443
```


### 0x0502 Spark Trouble Shooting

1. 在配置Spark时，使用了Spark-without-hadoop版本，导致Spark启动时，找不到某些class而失败。失败的输出信息如下所示：

```bash
hadoop-master:~/spark-2.4.4-bin-without-hadoop/sbin$ ./start-all.sh
starting org.apache.spark.deploy.master.Master, logging to /home/dev/spark-2.4.4-bin-without-hadoop/logs/spark-ssldev-org.apache.spark.deploy.master.Master-1-hadoop-master.out
failed to launch: nice -n 0 /home/dev/spark-2.4.4-bin-without-hadoop/bin/spark-class org.apache.spark.deploy.master.Master --host hadoop-master --port 7077 --webui-port 8080
  	at java.lang.Class.getMethod0(Class.java:3018)
  	at java.lang.Class.getMethod(Class.java:1784)
  	at sun.launcher.LauncherHelper.validateMainClass(LauncherHelper.java:544)
  	at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:526)
  Caused by: java.lang.ClassNotFoundException: org.slf4j.Logger
  	at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
  	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
  	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:349)
  	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
  	... 7 more
full log in /home/dev/spark-2.4.4-bin-without-hadoop/logs/spark-dev-org.apache.spark.deploy.master.Master-1-hadoop-master.out
```

解决办法：使用Spark-with-Hadoop的版本。


2. Install Python3版本配套的pip3：

```bash
sudo python3 get-pip.py
```

验证：

```bash
# 出现新的问题
$ pip3 -V
Traceback (most recent call last):
  File "/usr/local/bin/pip3", line 6, in <module>
    from pip._internal.main import main
ModuleNotFoundError: No module named 'pip._internal'
```

解决：[pip错误 ImportError: No module named _internal](https://www.jianshu.com/p/be16892b8231)

```bash
sudo python3 get-pip.py --force-reinstall
```

验证pip安装：

```bash
$ pip3 -V
pip 19.3.1 from /usr/local/lib/python3.6/dist-packages/pip (python 3.6)
```

Install pyspark:

```bash
sudo -H pip3 install pyspark
```

Spark的python版本问题解决方案：

```bash
vim /etc/profile
#设置Spark的Python版本
export PYSPARK_PYTHON="/usr/bin/python3"
```

修改Spark python启动脚本文件：

```bash
# Configuration pyspark
cd spark-2.4.4-bin-hadoop2.6/bin
vim pyspark
```

修改文件中的`PYSPARK_PYTHON=python`为`PYSPARK_PYTHON=python3`，如下所示：

```bash
# Determine the Python executable to use for the executors:
if [[ -z "$PYSPARK_PYTHON" ]]; then
  if [[ $PYSPARK_DRIVER_PYTHON == *ipython* && ! $WORKS_WITH_IPYTHON ]]; then
    echo "IPython requires Python 2.7+; please install python2.7 or set PYSPARK_PYTHON" 1>&2
    exit 1
  else
    PYSPARK_PYTHON=python3
  fi
fi
export PYSPARK_PYTHON
```

## Reference

1. [Hadoop Cluster Setup](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)
2. [Hadoop: Setting up a Single Node Cluster](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Fully-Distributed_Operation)
3. [ubuntu18.04 server配置ssh免密登录](https://blog.csdn.net/CowBoySoBusy/article/details/86558017)
4. [Hadoop完全分布式集群搭建（2.9.0）](https://blog.csdn.net/sjmz30071360/article/details/79889055)
5. [安装和配置Hadoop集群(3节点)](https://cloud.tencent.com/developer/article/1350441)
6. [将 Apache Spark 和 Apache Hive 与 Hive 仓库连接器相集成](https://docs.azure.cn/zh-cn/hdinsight/interactive-query/apache-hive-warehouse-connector)
7. [HiveDerbyServerMode](https://cwiki.apache.org/confluence/display/Hive/HiveDerbyServerMode)
8. [AdminManual Configuration](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Configuration)
9. [AdminManual Installation](https://cwiki.apache.org/confluence/display/Hive/AdminManual+Installation#AdminManualInstallation-InstallingfromaTarball)
10. [apache-hive-1.2.1-bin 安装](https://blog.csdn.net/thinktothings/article/details/85109254)
11. [HIve安装](https://www.jianshu.com/p/f745ed6a8e27)
12. [Ubuntu18.04 安装MySQL](https://blog.csdn.net/weixx3/article/details/80782479)
13. [ubuntu上mysql允许远程连接](https://www.jianshu.com/p/ee3f8b3a28bd)
14. [启动hive hiveserver2会报警告-Mon Oct 16 10:25:12 CST 2017 WARN: Establishing SSL connection without server](https://blog.csdn.net/helloxiaozhe/article/details/78252135)
15. [Hive篇.连接mysql报错The reference to entity "useSSL" must end with the ';' delimiter](https://blog.csdn.net/lwf006164/article/details/96195213)
16. [开启MySQL远程访问权限 允许远程连接](https://www.cnblogs.com/weifeng1463/p/7941625.html)
17. [Apache Hive GettingStarted](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-RunningHiveServer2andBeeline)
18. [配置项说明：Apache Hive Configuration Properties](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-HiveServer2)
19. [Apache Hive Setting Up HiveServer2](https://cwiki.apache.org/confluence/display/Hive/Setting+Up+HiveServer2)
20. [Hive DDL官方指南：LanguageManual DDL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)
21. [Hive DML官方指南：Hive Data Manipulation Language](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML)
22. [Spark: Running Spark on YARN](http://spark.apache.org/docs/latest/running-on-yarn.html#configuration)
23. [Kafka Quickstart](https://kafka.apache.org/quickstart)
24. [ZooKeeper Getting Started Guide](http://zookeeper.apache.org/doc/current/zookeeperStarted.html)
25. [LanguageManual SortBy](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SortBy)
26. [hadoop3.0全分布式集群搭建](https://www.lousenjay.top/2018/08/21/hadoop3.0%E5%85%A8%E5%88%86%E5%B8%83%E5%BC%8F%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/)
27. [ERROR 1819 (HY000): Your password does not satisfy the current policy requirements](https://www.cnblogs.com/ivictor/p/5142809.html)
