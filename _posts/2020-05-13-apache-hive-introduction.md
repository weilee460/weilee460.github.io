---
layout:     post
title:     Apache Hive Simple Use
author:   风止
category:  blog
description: 由于项目需要用到分布式数据库Hive，因此需要初步了解一下Hive。该blog的主要思路是：Hive是什么？Hive的特点？Hive数据库与数据表等使用方式？以及使用方式中的一些基本概念等。
---


## 0x00 Introduction

由于项目需要存储的数据量很大（需要存储长达数年的数据），因此传统的MySQL等数据库，无法满足此种应用场景。因此考虑使用分布式的数据库，便于处理大量的数据，并且在用户需求变化（例如用户需要存储更长时间的数据）时，可以方便的扩展数据库的存储。经过讨论，选择使用Hive作为数据库，因为项目应用场景对于实时性的要求并不高。这里记录一下Hive的初步学习了解。

## 0x01 What's Hive

Hive官网对Hive描述如下，来源于参考[2]：

> The Apache Hive™ data warehouse software facilitates reading, writing, and managing large datasets residing in distributed storage and queried using SQL syntax.
> 
> Built on top of Apache Hadoop™, Hive provides the following features:
> 
> * Tools to enable easy access to data via SQL, thus enabling data warehousing tasks such as extract/transform/load (ETL), reporting, and data analysis.
> * A mechanism to impose structure on a variety of data formats
> * Access to files stored either directly in Apache HDFS™ or in other data storage systems such as Apache HBase™ 
> * Query execution via Apache Tez™, Apache Spark™, or MapReduce
> * Procedural language with HPL-SQL
> * Sub-second query retrieval via Hive LLAP, Apache YARN and Apache Slider.

> Hive provides standard SQL functionality, including many of the later SQL:2003, SQL:2011, and SQL:2016 features for analytics.
> Hive's SQL can also be extended with user code via user defined functions (UDFs), user defined aggregates (UDAFs), and user defined table functions (UDTFs).
> 
> There is not a single "Hive format" in which data must be stored. Hive comes with built in connectors for comma and tab-separated values (CSV/TSV) text files, Apache Parquet™, Apache ORC™, and other formats. Users can extend Hive with connectors for other formats. Please see File Formats and Hive SerDe in the Developer Guide for details.
>
> Hive is not designed for online transaction processing (OLTP) workloads. It is best used for traditional data warehousing tasks.
>
> Hive is designed to maximize scalability (scale out with more machines added dynamically to the Hadoop cluster), performance, extensibility, fault-tolerance, and loose-coupling with its input formats.

简而言之：Hive是一个数据仓库软件，主要的用途是在大规模的分布式数据存储中，**以SQL方式执行数据查询等操作**。Hive最适用的场景是应用于传统的数据仓库任务，不适用于在线交易处理（OLTP）类的任务。直白点说， **Hive不适用于实时性要求较高的场景。**

**Hive的本质** 可以说是一个**解释器**，将SQL语句解释为MapReduce来执行数据操作。并非实传统意义上的数据库。

两个名词说明：

* **ETL:**  extract/transform/load. 数据的抽取、转换和加载。
* **OLTP:**  online transaction processing. 在线交易处理（or 联机事务处理）。

行业中，数据处理分为两大类：

* OLTP(online transaction processing)：联机事务处理。
* OLAP(online analytical processing)：联机分析处理。

**Hive主要用途：**

作为实时性要求较低的场景中的一个数据仓库。例如用于离线的数据分析，以支持商务决策等。不支持在线的数据分析用途。例如在线实时分析用户的点击情况，并推送适当的推荐信息（or 广告）给用户。

## 0x02 行存储与列存储

**存储方式方面** 见下图，来源于参考[11]：

![](https://img-blog.csdn.net/20180706125239844?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpbmd4aW54aW54aW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![](/images/20180706125239844.png)

一图胜千言，通俗点说：行式存储中，一张表的数据全部存储在一起，表中一行的字段是相邻存储的；列式存储中，一张表的数据是按字段分开存储的，表中同一列的数据是相邻存储的，表中同一行的数据是分开存储的。

**存储数据压缩方面**，来源于参考[11]：

![](https://img-blog.csdn.net/20180706125401662?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpbmd4aW54aW54aW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

简而言之：当列中的取值只有有限的几种，此时只需要将几种取值单独使用一张表存储（称为取值表），而原表中的该列的取值则使用取值表的索引来表示。此时无需将每一个取值均存储一遍，大幅的减少存储的数据量，尤其当数据量是海量的时候。

**查询执行方式**，来源于参考[11]：

![](https://img-blog.csdn.net/20180706125430267?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1hpbmd4aW54aW54aW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

简而言之：没看懂，待补充。 :-) :-)

**两种存储方式的特点：**
 
 * 行式存储：由于同一行的数据是相邻存储的，因此按照行抽取数据时，效率较高。
 
 * 列式存储：由于通一列的数据是相邻存储的，因此按照列抽取数据时，效率较高。尤其适用于抽取数据中特定的列。此外由于同一列数据类型是相同的，因此可以针对不同类型的数据运用不同的压缩算法，提高存储的效率。


## 0x03 基本概念

数据库创建， 来源于Hive官网：

> Create Database
> 
```SQL
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
  [COMMENT database_comment]
  [LOCATION hdfs_path]
  [MANAGEDLOCATION hdfs_path]
  [WITH DBPROPERTIES (property_name=property_value, ...)];
```

`IF NOT EXISTS`可以确保数据库创建时，Hive中已有该数据库，则不会再次创建。防止重复创建的问题。

数据表创建， 来源于Hive官网：

> Create Table
> 
```SQL
CREATE [TEMPORARY] [EXTERNAL] TABLE [IF NOT EXISTS] [db_name.]table_name    -- (Note: TEMPORARY available in Hive 0.14.0 and later)
  [(col_name data_type [column_constraint_specification] [COMMENT col_comment], ... [constraint_specification])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [CLUSTERED BY (col_name, col_name, ...) [SORTED BY (col_name [ASC|DESC], ...)] INTO num_buckets BUCKETS]
  [SKEWED BY (col_name, col_name, ...)                  -- (Note: Available in Hive 0.10.0 and later)]
     ON ((col_value, col_value, ...), (col_value, col_value, ...), ...)
     [STORED AS DIRECTORIES]
  [
   [ROW FORMAT row_format] 
   [STORED AS file_format]
     | STORED BY 'storage.handler.class.name' [WITH SERDEPROPERTIES (...)]  -- (Note: Available in Hive 0.6.0 and later)
  ]
  [LOCATION hdfs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]   -- (Note: Available in Hive 0.6.0 and later)
  [AS select_statement];   -- (Note: Available in Hive 0.5.0 and later; not supported for external tables)
```

其中数据表的创建，有几个重要的概念：EXTERNAL，PARTITIONED，BUCKETS， STORED AS等。这里说明一下EXTERNAL，PARTITIONED，以及 STORED AS。

### 0x0301 Managed Table vs External Table

Hive官网对Managed(Internal) Table和External Table的描述为：

> * Managed tables
> 
>> A managed table is stored under the hive.metastore.warehouse.dir path property, by default in a folder path similar to /user/hive/warehouse/databasename.db/tablename/. The default location can be overridden by the location property during table creation. If a managed table or partition is dropped, the data and metadata associated with that table or partition are deleted. If the PURGE option is not specified, the data is moved to a trash folder for a defined duration.
>>
>> Use managed tables when Hive should manage the lifecycle of the table, or when generating temporary tables.
>
> * External tables
> 
>> An external table describes the metadata / schema on external files. External table files can be accessed and managed by processes outside of Hive. External tables can access data stored in sources such as Azure Storage Volumes (ASV) or remote HDFS locations. If the structure or partitioning of an external table is changed, an MSCK REPAIR TABLE table_name statement can be used to refresh metadata information.
>> 
>> Use external tables when files are already present or in remote locations, and the files should remain even if the table is dropped.

简答的说：Managed Table由Hive管理，管理该table的存储位置等等；而External Table由用户管理，可以指定存储的位置等等。此外External Table在删除时，数据文件并不会从HDFS中删除，需要用户自己删除。

那存储数据时，如何选择Managed Table和External Table呢？这里参考[Hive 之Table、External Table、Partition(五)](https://blog.csdn.net/u013850277/article/details/65749770)，hadoop权威指南3如下建议:

![](https://img-blog.csdn.net/20170325000325569?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMzg1MDI3Nw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Hive对两种类型的表描述为：

> Hive fundamentally knows two different types of tables:
>
> * Managed (Internal)
> * External

注意：这里个人觉得Internal和External更合适，翻译为内部表和外部表。

### 0x0302 Table Format

Hive支持的存储格式如下，来源于Hive官网：

| storage format | Description |
| -------------- | ----------- |
| STORED AS TEXTFILE | Stored as plain text files. TEXTFILE is the default file format, unless the configuration parameter hive.default.fileformat has a different setting. Use the DELIMITED clause to read delimited files.Enable escaping for the delimiter characters by using the 'ESCAPED BY' clause (such as ESCAPED BY '\') Escaping is needed if you want to work with data that can contain these delimiter characters. A custom NULL format can also be specified using the 'NULL DEFINED AS' clause (default is '\N'). |
| STORED AS SEQUENCEFILE | Stored as compressed Sequence File. |
| STORED AS RCFILE | Stored as Record Columnar File format. |
| STORED AS PARQUET | Stored as Parquet format for the Parquet columnar storage format in Hive 0.13.0 and later; Use ROW FORMAT SERDE ... STORED AS INPUTFORMAT ... OUTPUTFORMAT syntax ... in Hive 0.10, 0.11, or 0.12.|
| STORED AS ORC | Stored as ORC file format. Supports ACID Transactions & Cost-based Optimizer (CBO). Stores column-level metadata.|
| STORED AS JSONFILE | Stored as Json file format in Hive 4.0.0 and later.|
| STORED AS AVRO | Stored as Avro format in Hive 0.14.0 and later (see Avro SerDe).|

从上表来看，Hive支持的数据表存储格式主要有：TEXTFILE，SEQUENCEFILE，RCFILE，PARQUET，ORC，JSONFILE，AVRO。这里主要说明一下TEXTFILE和PARQUET。TEXTFILE格式就是普通的文本。因此下面说一下PARQUET格式。

**PARQUET**

Hive官方文档对于Parquet如下描述，来源于[Parquet](https://cwiki.apache.org/confluence/display/Hive/Parquet)：

> Parquet (http://parquet.io/) is an ecosystem wide columnar format for Hadoop. Read Dremel made simple with Parquet for a good introduction to the format while the Parquet project has an in-depth description of the format including motivations and diagrams.

简而言之：Parquet就是一个列存储格式，用于Hadoop中。

使用Parquet的目的是：
> **Parquet Motivation**
>
> We created Parquet to make the advantages of compressed, efficient columnar data representation available to any project in the Hadoop ecosystem.
>
> Parquet is built from the ground up with complex nested data structures in mind, and uses the record shredding and assembly algorithm described in the Dremel paper. We believe this approach is superior to simple flattening of nested name spaces.
>
> Parquet is built to support very efficient compression and encoding schemes. Multiple projects have demonstrated the performance impact of applying the right compression and encoding scheme to the data. Parquet allows compression schemes to be specified on a per-column level, and is future-proofed to allow adding more encodings as they are invented and implemented.
> 
> Parquet is built to be used by anyone. The Hadoop ecosystem is rich with data processing frameworks, and we are not interested in playing favorites. We believe that an efficient, well-implemented columnar storage substrate should be useful to all frameworks without the cost of extensive and difficult to set up dependencies.

简而言之：使用Parquet的目的是利用其压缩性能好等等特点。

从[Hive 列存储简介](http://icejoywoo.github.io/2016/03/29/hive-ocr-and-parquet.html)blog可知：Parquet的压缩性能非常不错。可以大规模节约存储。

### 0x0303 Partition and Bucket

数据表分区分桶：待补充。

Hive官网对于Partition的描述如下：

> Partitioned Tables
> 
>> Partitioned tables can be created using the PARTITIONED BY clause. A table can have one or more partition columns and a separate data directory is created for each distinct value combination in the partition columns. Further, tables or partitions can be bucketed using CLUSTERED BY columns, and data can be sorted within that bucket via SORT BY columns. This can improve performance on certain kinds of queries.
>>
>> If, when creating a partitioned table, you get this error: "FAILED: Error in semantic analysis: Column repeated in partitioning columns," it means you are trying to include the partitioned column in the data of the table itself. You probably really do have the column defined. However, the partition you create makes a pseudocolumn on which you can query, so you must rename your table column to something else (that users should not query on!).


## 0x04 Hive Install

Hive基本安装配置，请参考[Hadoop Cluster Configuration](https://weilee460.github.io/hadoop-cluster-configuration)


## 0x05 Data Manipulation

hive shell login：

```bash
$ hive
```

查看数据库，并选择数据库，查看数据表：

```SQL
hive>show databases;

hive> use testdb;

hive> show tables;
```

查询数据，限制行数：

```SQL
hive> select * from test_table limit 30;
```

查询数据总数：

```SQL
hive>select count(*) from test_table;

...     ...
Total MapReduce CPU Time Spent: 27 seconds 760 msec
OK
11724128
Time taken: 33.982 seconds, Fetched: 1 row(s)
```

查询数据时间范围，即查询时间的最大值和最小值：

```SQL
hive>select min(info_time) from test_table;

...      ....
Total MapReduce CPU Time Spent: 52 seconds 930 msec
OK
2003-10-27 19:28:34
Time taken: 42.831 seconds, Fetched: 1 row(s)
```

```SQL
hive>select max(info_time) from test_table;

...     ...
Stage-Stage-1: Map: 4  Reduce: 1   Cumulative CPU: 54.01 sec   HDFS Read: 849634116 HDFS Write: 119 SUCCESS
Total MapReduce CPU Time Spent: 54 seconds 10 msec
OK
2018-11-14 11:01:50
Time taken: 50.997 seconds, Fetched: 1 row(s)
```

抽取特定时间范围，特定列（假设列名是test_col1）的数据：

```SQL
hive> select test_col1 from test_table where info_time>'2002-8-9 23:09:21' and info_time<'2012-8-9 23:09:21';
... ...

```

数据表中插入数据，该数据按照年月日分区，表中有两列：

```SQL
hive> INSERT INTO test_table PARTITION (y=2020, m=12, d=12) VALUES (12, 'test_name', 'test_char');
```

以load方式在数据表中添加数据，数据文件位于本地目录内：

```SQL
hive> load data local inpath '/test_data/2003-10-27.csv' into table test_table partition (y=2003, m=10, d=27);
```


## 0x06 Hive Server

remote数据入库的应用场景如下：

* 入库数据的来源是csv文件；
* 入库数据需要按照年、月、日分区；

两种数据入库的方式，分别使用insert和load语句将数据存入数据库。

```python
from pyhive import hive

# create database connect
hive_host = server_ip
hive_port = server_port
hive_username = username
hive_dbname = "testdb"
hive_auth = "NOSASL"

conn = hive.connect(host=hive_host, port=hive_port,  username=hive_username, database=hive_dbname, auth=hive_auth)
cursor = conn.cursor()
  
sql_cmd = "INSERT INTO test_table2 PARTITION (y={0}, m={1}, d={2}) VALUES (data1, data2, data3, data4, data5)".format(2003, 10, 27)

cursor.execute(sql_cmd)

# close database resource
cursor.close()
conn.close()
```

注意： 若`Insert`语句中的`VALUES`的某些字段是字符串时，需要带有单引号。例如`sql_cmd = "INSERT INTO test_table2 PARTITION (y=2003, m=10, d=27) VALUES ('{0}', '{1}', '{2}', '{3}', '{4}')".format(str1, str2, str3, str4, str5)
`
以load方式添加数据到Hive中：

```python
from pyhive import hive

# create database connect
hive_host = server_ip
hive_port = server_port
hive_username = username
hive_dbname = "testdb"
hive_auth = "NOSASL"

conn = hive.connect(host=hive_host, port=hive_port,  username=hive_username, database=hive_dbname, auth=hive_auth)
cursor = conn.cursor()

sql_cmd = "LOAD DATA INPATH '/test_data/2003-10-27.csv' INTO TABLE test_table PARTITION (y=2003, m=10, d=27)"

cursor.execute(sql_cmd)

# close database resource
cursor.close()
conn.close()
```

注意⚠️：上述`load`语句中的`inpath`指向的文件为hdfs文件系统中的文件。这里不能使用`local inpath`，除非数据位于hivesever设备中的存储中，而这在生产环境中，往往是不可能的。

使用load方式进行数据入库时，其执行方式：客户端连接hive server，hive server接收客户端发送的SQL命令并解释执行。因此，当load本地数据到hive中时，hive server解析load中的本地路径时，会将数据路径中的本地路径解释为hive server中的本地路径。若hive server没有需要入库的本地数据（客户端与hive server不是同一台设备），将会报错“在路径中找不到文件”。为解决次问题，需要将客户端的本地数据放入hive server中（生产环境中通常是不可能的），或者将数据存储到hdfs中，在hive load数据时使用hdfs的路径（本次采用这种方式）。


## Reference

1. [Hive GettingStarted](https://cwiki.apache.org/confluence/display/Hive/GettingStarted#GettingStarted-DMLOperations)
2. [Apache Hive](https://cwiki.apache.org/confluence/display/Hive/Home)
3. [LanguageManual Select](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Select)
4. [LanguageManual SortBy](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+SortBy)
5. [处理海量数据：列式存储综述（存储篇）](https://zhuanlan.zhihu.com/p/35622907)
6. [MongoDB、ElasticSearch、Redis、HBase这四种热门数据库的优缺点及应用场景](https://zhuanlan.zhihu.com/p/37964096)
7. [处理海量数据：列式存储综述（系统篇）](https://zhuanlan.zhihu.com/p/38224411)
8. [Parquet与ORC：高性能列式存储格式(收藏)](https://www.cnblogs.com/wujin/p/6208734.html)
9. [Hive执行load data [local] inpath 'path' [overwrite ] into table table_name报Invalid path问题](https://blog.csdn.net/hao495430759/article/details/80529456)
10. [Hive 之Table、External Table、Partition(五)](https://blog.csdn.net/u013850277/article/details/65749770)
11. [行存储 VS 列存储](https://blog.csdn.net/Xingxinxinxin/article/details/80939277)
12. [Parquet](https://cwiki.apache.org/confluence/display/Hive/Parquet)
13. [Hive 列存储简介](http://icejoywoo.github.io/2016/03/29/hive-ocr-and-parquet.html)

