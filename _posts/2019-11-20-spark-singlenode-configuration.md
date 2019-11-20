
---
layout:     post
title:      Spark Single Node Configuration and PrefixSpan Algorithm Test
category: blog
description: 网络安全领域由于各种网络安全设备产生的日志信息较多，网络管理人员人工查看这些信息是无法接受的，因此常常建立一个SOC（安全运营中心）来处理各种信息。SOC其实是传统网络安全方案中的SIEM的衍生，不仅处理安全信息，进一步使用数据挖掘技术等，对安全信息进行分析，勾画当前网络环境安全的整体状况。SOC常见的落地方案是建立在Hadoop生态圈上。
---

## 0x00 Overview

近期，由于公司`IDS`类产品的告警信息太多，因此需要一个`SIEM`类平台，来集中处理这些告警信息。此外，当前的告警信息需要与以前的告警信息关联起来，来展现某些设备经常遭受的网络攻击；甚至进一步展现防护的资产所遭受网络攻击情况，便于资产所有方对其资产面临的安全状况有一个全面点的了解。

把每一条告警信息按照序列数据来理解，那么上述的需求，就演变为了序列数据的关联分析。此时，数据挖掘中的关联分析技术就派上用场了。数据挖掘中的关联分析算法`PrefixSpan`是常用的算法，因此测试一下该算法的关联分析效果。

在大数据分析中，常用的是“Hadoop生态圈”中的技术。例如，使用`Hadoop`来存储数据，使用`Spark`对数据进行计算分析。此外，`Spark`还自带了对"Machine Learning"的算法的支持，即`Spark MLlib`。`MLlib`中包含了"PrefixSpan"算法。因此考虑使用Spark来测试。

## 0x01 OS Configuration

由于仅仅是测试使用，因此考虑搭建一个单节点的Spark计算引擎。OS选取大家常用的`Ubuntu 18.04LTS`。系统安装好，需要安装开发所需的软件包，例如：`sudo apt-get install build-essential`等。

这里Java选取的版本为：`jdk1.8.0_191`。

解压后，得到`jdk1.8.0_191`目录，配置环境变量即可。

个人配置如下：

```
# .profile文件是Ubuntu中默认的用户环境变量配置文件
vim .profile

#add rows as follows, modify java home directory
export JAVA_HOME="xxx/jdk1.8.0_191"
export PATH="$JAVA_HOME/bin:$PATH"
export CLASSPATH=".:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar"
```

验证：

```
$ java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```

## 0x02 Spark Configuration

首先，配置Spark支持Java的计算环境；然后配置Spark支持Python的计算环境。

### 0x0200 Spark Java Environment Configuration

Download Spark, 得到`spark-2.4.4-bin-hadoop2.6.tgz`，并解压。

Create configuration files:

```
cd spark-2.4.4-bin-hadoop2.6/conf

cp spark-env.sh.template spark-env.sh
cp spark-defaults.conf.template spark-defaults.conf
cp slaves.template slaves
```

Configuration:

```
vim spark-env.sh
# add configurations, Set Your IP Address, and Spark home directory
SPARK_LOCAL_IP=192.168.x.xxx
export SPARK_HOME=xxx/spark-2.4.4-bin-hadoop2.6
```

Test and Verification:

```
cd xxx/spark-2.4.4-bin-hadoop2.6/examples/src/main/scala/org/apache/spark/examples
$ xxx/spark-2.4.4-bin-hadoop2.6/bin/run-example SparkPi 10

19/11/20 11:09:25 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
19/11/20 11:09:25 INFO SparkContext: Running Spark version 2.4.4
19/11/20 11:09:25 INFO SparkContext: Submitted application: Spark Pi

... ...

19/11/20 11:09:27 INFO DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 0.756038 s
Pi is roughly 3.144059144059144
19/11/20 11:09:27 INFO SparkUI: Stopped Spark web UI at http://192.168.x.xxx:4040
```

### 0x0201 Spark Python Environment Configuration

Ubuntu 18.04LTS自带了两个Python版本。分别是：

```
$ python -V
Python 2.7.15+

$ python3 -V
Python 3.6.8
```

这里选择了Python3.6.8。Install pyspark:

```
sudo pip3 install pyspark
```

Set PYSPARK_PYTHON value:

```
vim ~/.profile
#add one line to file end
export PYSPARK_PYTHON=/usr/bin/python3
```

Modify spark-2.4.4-bin-hadoop2.6/bin/pyspark file:

```
vim spark-2.4.4-bin-hadoop2.6/bin/pyspark
```

源文件部分内容如下：

```
# Determine the Python executable to use for the executors:
if [[ -z "$PYSPARK_PYTHON" ]]; then
  if [[ $PYSPARK_DRIVER_PYTHON == *ipython* && ! $WORKS_WITH_IPYTHON ]]; then
    echo "IPython requires Python 2.7+; please install python2.7 or set PYSPARK_PYTHON" 1>&2
    exit 1
  else
    PYSPARK_PYTHON=python
  fi
fi
export PYSPARK_PYTHON
```

修改为，注意其中的python版本：

```
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

修改spark-env.sh：

```
vim spark-env.sh
# add one line to file end
export PYSPARK_PYTHON=/usr/bin/python3
```

Test and Verification:

```
$cd spark-2.4.4-bin-hadoop2.6/bin

$pyspark

output:
Python 3.6.8 (default, Oct  7 2019, 12:59:55)
[GCC 8.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
19/11/20 14:51:07 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.4.4
      /_/

Using Python version 3.6.8 (default, Oct  7 2019 12:59:55)
SparkSession available as 'spark'.
```

Attention: Python version is V3.6.8, not Python 2.7.15!!

此时，Spark支持Python的计算环境配置完成。

## 0x03 PrefixSpan Demo

测试Spark官网提供的example：

```
from pyspark.ml.fpm import PrefixSpan

df = sc.parallelize([Row(sequence=[[1, 2], [3]]),
                     Row(sequence=[[1], [3, 2], [1, 2]]),
                     Row(sequence=[[1, 2], [5]]),
                     Row(sequence=[[6]])]).toDF()

prefixSpan = PrefixSpan(minSupport=0.5, maxPatternLength=5,
                        maxLocalProjDBSize=32000000)

# Find frequent sequential patterns.
prefixSpan.findFrequentSequentialPatterns(df).show()
```

执行将会报错：sc not found. 即代码中的sc对象没有发现。

使用参考[4]中的`sc`对象生成代码，添加：

```
sc = SparkContext("local","testing")
```

执行报错：

```
AttributeError: 'RDD' object has no attribute 'toDF'
```

检查example中的代码，发现在`df`对象生成语句的最后，调用了`toDF()`接口。据此，猜测由于Spark的版本升级，接口可能发生了变化。好在Spark提供了较为完整的example做参考。

参考`spark-2.4.4-bin-hadoop2.6/examples/src/main/python/ml/prefixspan_example.py`，`sc`对象的生成修改为：

```
    spark = SparkSession\
        .builder\
        .appName("PrefixSpanTest")\
        .getOrCreate()

    sc = spark.sparkContext
```

修改后的完成的代码如下：

```
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
    spark = SparkSession\
        .builder\
        .appName("PrefixSpanTest")\
        .getOrCreate()

    sc = spark.sparkContext

    df = sc.parallelize([Row(sequence=[[1, 2], [3]]),
                     Row(sequence=[[1], [3, 2], [1, 2]]),
                     Row(sequence=[[1, 2], [5]]),
                     Row(sequence=[[6]])]).toDF()

    prefixSpan = PrefixSpan(minSupport=0.5, maxPatternLength=5,
                        maxLocalProjDBSize=32000000)

    # Find frequent sequential patterns.
    prefixSpan.findFrequentSequentialPatterns(df).show()
```

excute: `python3 spark_demo.py`

程序输出如下：

```
19/11/20 14:13:11 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
19/11/20 14:13:17 WARN PrefixSpan: Input data is not cached.
+----------+----+
|  sequence|freq|
+----------+----+
|     [[3]]|   2|
|     [[2]]|   3|
|     [[1]]|   3|
|  [[1, 2]]|   3|
|[[1], [3]]|   2|
+----------+----+
```

Succ!!   :-) :-)

## Reference

1. [Spark MLlib](https://spark.apache.org/mllib/)
2. [pyspark设置python的版本](https://blog.csdn.net/abc_321a/article/details/82589836)
3. [Frequent Pattern Mining](https://spark.apache.org/docs/latest/ml-frequent-pattern-mining.html)
4. [用Spark学习FP Tree算法和PrefixSpan算法](https://www.cnblogs.com/pinard/p/6340162.html)