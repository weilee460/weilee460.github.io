---
layout:     post
title:     Spark Simple Use(Scala)
author:   风止
category: blog
description: 由于项目需要，这里记录一下使用Scala开发Spark应用的简单用法等等。
---

## 0x00 Introduction

由于项目需要，使用Spark作为数据分析的计算引擎，使用Python开发了数据分析的应用。但在测试中发现，满足计算需求所需的设备较多，配置要更高。因此为较少项目的设备成本，考虑使用Spark原生的语言Scala开发计算应用。因此使用Scala开发Spark的应用。


## 0x01 Scala Simple Syntax

Scala简单语法使用，参考来源为：[Scala 教程](https://www.runoob.com/scala/scala-tutorial.html)

Scala数据类型有：

| 数据类型 | 描述 |
| ------ | ---- |
| Byte | 8位有符号补码整数。数值区间为 -128 到 127 |
| Short | 16位有符号补码整数。数值区间为 -32768 到 32767 |
| Int | 32位有符号补码整数。数值区间为 -2147483648 到 2147483647 |
| Long | 64位有符号补码整数。数值区间为 -9223372036854775808 到 9223372036854775807 |
| Float | 32 位, IEEE 754 标准的单精度浮点数 |
| Double | 64 位 IEEE 754 标准的双精度浮点数 |
| Char | 16位无符号Unicode字符, 区间值为 U+0000 到 U+FFFF |
| String | 字符序列 |
| Boolean | true或false |
| Unit | 表示无值，和其他语言中void等同。用作不返回任何结果的方法的结果类型。Unit只有一个实例值，写成()。|
| Null | null 或空引用 |
| Nothing | Nothing类型在Scala的类层级的最底端；它是任何其他类型的子类型。|
| Any | Any是所有其他类的超类|
| AnyRef | AnyRef类是Scala里所有引用类(reference class)的基类 |

Scala变量申明：

*  声明变量，使用关键词"var"。
*  声明常量，使用关键词"val"。

Sample:

```
// 变量，类型是String
var testVar : String = "test"

// 常量，类型是String
val testVal : String = "test"
```

Scala for 循环：

```
for( var x <- Range ){
   statement(s);
}
```

> 以上语法中，Range 可以是一个数字区间表示 `i to j`（闭区间），或者 `i until j`（前闭后开区间）。左箭头`<-`用于为变量 x 赋值。

Scala for遍历集合：

```
for( x <- List ){
   statement(s);
}
```
> 以上语法中， List 变量是一个集合，for 循环会迭代所有集合的元素。

Scala while 循环语法：

```
while(condition)
{
   statement(s);
}
```

Scala do while 循环语法：

```
do {
   statement(s);
} while( condition );
```

Scala 方法定义：

```
def functionName ([参数列表]) : [return type] = {
   function body
   return [expr]
}
```

Scala数组：

```
var z:Array[String] = new Array[String](3)
```

Scala Collection:

| 序号 | 集合及描述 |
| --- | -------- |
| 1 | Scala List：List的特征是其元素以线性方式存储，集合中可以存放重复对象 |
| 2 | Scala Set(集合)：Set是最简单的一种集合。集合中的对象不按特定的方式排序，并且没有重复对象。|
| 3 | Scala Map(映射)：Map 是一种把键对象和值对象映射的集合，它的每一个元素都包含一对键对象和值对象。|
| 4 | Scala 元组：元组是不同类型的值的集合 |
| 5 | Scala Option：Option[T] 表示有可能包含值的容器，也可能不包含值。|
| 6 | Scala Iterator（迭代器）：迭代器不是一个容器，更确切的说是逐一访问容器内元素的方法。|


Seq:

[Scala Seq用法示例](http://www.srcmini.com/34680.html)

注：在Spark中的Array与Scala中的Array并不等价，Scala中的Array数据需要转换为Seq类型的。有点坑 :-) :-)

## 0x02 Spark Scala

### 0x0200 Spark 内置的数据类型

参考来源：

1. [Spark SQL数据类型](https://www.w3cschool.cn/spark/3x1rgozt.html)
2. [Spark SQL：怎样修改DataFrame列的数据类型？](https://zhuanlan.zhihu.com/p/148648985)
3. [Spark Data Types](http://spark.apache.org/docs/latest/sql-ref-datatypes.html)
4. [Spark Functions](http://spark.apache.org/docs/latest/sql-ref-functions.html)

直接参考官方说明即可，这里不细说。

### 0x0201 Spark Important Class

数据类型官方说明文档(Spark V2.4.4)：

1. [org.apache.spark.sql.Row](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.sql.Row)
2. [org.apache.spark.sql.Dataset](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.sql.Dataset)
3. [org.apache.spark.SparkConf](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.SparkConf)
4. [org.apache.spark](http://spark.apache.org/docs/2.4.4/api/scala/#org.apache.spark.package)
5. [org.apache.spark.SparkContext](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.SparkContext)
6. [org.apache.spark.rdd.RDD](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.rdd.RDD)
7. [org.apache.spark.sql](http://spark.apache.org/docs/2.4.4/api/scala/#org.apache.spark.sql.package)
8. 


数据类型官方说明文档(Spark V3.0.0)：

1. [org.apache.spark.rdd.RDD](http://spark.apache.org/docs/latest/api/scala/org/apache/spark/rdd/RDD.html)
2.  [org.apache.spark.sql.Dataset](http://spark.apache.org/docs/latest/api/scala/org/apache/spark/sql/Dataset.html)
3. []()

### 0x0202 Spark Simple Demo

参考来源：

1. [使用Spark对数据进行分组排序（Java和Scala实现）](https://blog.csdn.net/u010592112/article/details/81032675)
2. [Spark分组取TopN](https://blog.csdn.net/sinat_36710456/article/details/86569441)
3. [Spark（三）-- SparkSQL扩展（数据操作） -- 聚合(四)](https://blog.csdn.net/qq_18800463/article/details/102621157)
4. [Spark SQL 数据统计 Scala 开发小结](https://cloud.tencent.com/developer/article/1005690)
5. [一文让你了解DataSet处理Sql的各种实战技巧](https://blog.csdn.net/lidongmeng0213/article/details/102939871)
6. 

#### 0x020200 DataFrame generate

读取csv文件中的数据，生成`DataFrame`：

```
var test_dataDF = spark.read.format("csv").option("header", "true")
      .load("your_data_path/2018-03-12_src.csv")

test_dataDF.printSchema()
```

DataFrame and Schema:

```bash
//dataframe

+-------------------+---------------+------------+------------+
|          info_time|             ip|incidents_id|     type_id|
+-------------------+---------------+------------+------------+
|2018-03-12 19:28:38| 192.168.199.17|     1      |           2|
|2018-03-13 19:28:38|192.168.197.215|     2      |           2|
|2018-03-14 19:28:38|192.168.195.125|     7      |           8|
+-------------------+---------------+------------+------------+

//schema

root
 |-- info_time: string (nullable = true)
 |-- dst_ip: string (nullable = true)
 |-- attck_tech: string (nullable = true)
 |-- attck_tactic: string (nullable = true)
```

#### 0x020201 DataFrame schema

DataFrame中数据类型转换：

可以看出，上述DataFrame的列数据类型默认是String类型。若想改变列的数据类型，有两种方式：一种是指定DataFrame的schema，另一种是使用转换的方式直接做转换。

```scala
      val new_schemaArray = Array(
        StructField("info_time", StringType, false),
        StructField("ip", StringType, false),
        StructField("incidents_id", IntegerType, false),
        StructField("type_id", IntegerType, false)
      )
      val new_schema = StructType(tacticID_schemaArray)
      val newSchema_mapDF = spark.createDataFrame(test_dataDF.rdd, new_schema)
      newSchema_mapDF.printSchema()
```

输出如下：

```
root
 |-- info_time: string (nullable = false)
 |-- dst_ip: string (nullable = false)
 |-- incidents_id: Integer (nullable = false)
 |-- type_id: Integer (nullable = false)
```

#### 0x020202 DataFrame join

列数据join：使用join算子将DataFrame中的某列与其他DataFrame中的列做关联。例如将公司员工工资表与员工的社保表做关联，关联依据是员工的工号。

```Scala
val employDF = employ_salaryDF.join(socialInsurance_dataDF, employ_ salaryDF("employ_id") === socialInsurance_dataDF("employ_id1"), "inner")
```

注意：join方式有inner, outer等。

#### 0x020203 Spark UDF

Spark UDF。使用udf，根据员工ID，在DataFrame中新增“社保费用”一列数据：

```Scala
  //udf definition。employ_map是以员工ID为key的map
  val transformFunc = udf((x:String) => employ_map(x))
  //udf register
  spark.udf.register("transformFunc", transformFunc.asNondeterministic())
  //udf call 
  val transform_DF = employ_salaryDF.withColumn("insurance_idata", callUDF("transformFunc", col("employ_id")))
```

注意：若函数的参数类型不是默认的，需要在注册时声明参数的类型。例如：

```Scala
spark.udf.register("testFunc_udf", test_func(_:Seq[Int]), returnType = IntegerType)
```

若使用这种方式注册，则会根据DataFrame中的列类型（Array），默认推导出数据参数类型是Array，进而导致执行错误。

```Scala
spark.udf.register("testFunc_udf", test_func(_), returnType = IntegerType)
```

参考：

1. [Spark DataFrame withColumn](https://sparkbyexamples.com/spark/spark-dataframe-withcolumn/)
2. [Spark SQL UDF (User Defined Functions)](https://sparkbyexamples.com/spark/spark-sql-udf/)
3. [Spark: Scalar User Defined Functions (UDFs)](http://spark.apache.org/docs/latest/sql-ref-functions-udf-scalar.html)

#### 0x020203 DataFrame Filter

数据过滤。将数据中某列数据（类型是Array）不符合条件的数据筛除掉。

```Scala
//filter by array length
val filtered_df = test_dataDF.filter(size($"sequence") > 4)
```

参考：

1. [Spark SQL Array functions complete list](https://sparkbyexamples.com/spark/spark-sql-array-functions/)

## 0x03 API Explaination

### 0x0300 DataFrame

Scala DataFrame类定义：

```scala
type DataFrame = Dataset[Row]
```

### 0x0301 Dataset

参考来源：[org.apache.spark.sql.Dataset](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.sql.Dataset)

> A Dataset is a strongly typed collection of domain-specific objects that can be transformed in parallel using functional or relational operations. Each Dataset also has an untyped view called a DataFrame, which is a Dataset of Row.
> 
> Operations available on Datasets are divided into transformations and actions. Transformations are the ones that produce new Datasets, and actions are the ones that trigger computation and return results. Example transformations include map, filter, select, and aggregate (groupBy). Example actions count, show, or writing data out to file systems.
>
> Datasets are "lazy", i.e. computations are only triggered when an action is invoked. Internally, a Dataset represents a logical plan that describes the computation required to produce the data. When an action is invoked, Spark's query optimizer optimizes the logical plan and generates a physical plan for efficient execution in a parallel and distributed manner. To explore the logical plan as well as optimized physical plan, use the explain function.
>
> To efficiently support domain-specific objects, an Encoder is required. The encoder maps the domain specific type T to Spark's internal type system. For example, given a class Person with two fields, name (string) and age (int), an encoder is used to tell Spark to generate code at runtime to serialize the Person object into a binary structure. This binary structure often has much lower memory footprint as well as are optimized for efficiency in data processing (e.g. in a columnar format). To understand the internal binary representation for data, use the schema function.

### 0x0302 Row

 参考来源：[org.apache.spark.sql.Row](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.sql.Row)

Row 官网说明：
 
> Represents one row of output from a relational operator. Allows both generic access by ordinal, which will incur boxing overhead for primitives, as well as native primitive access.
>
> It is invalid to use the native primitive interface to retrieve a value that is null, instead a user must check isNullAt before attempting to retrieve a value that might be null.
>
> To create a new Row, use RowFactory.create() in Java or Row.apply() in Scala.


### 0x0303 RDD

参考来源：[org.apache.spark.rdd.RDD](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.rdd.RDD)

RDD官网说明：

> A Resilient Distributed Dataset (RDD), the basic abstraction in Spark. Represents an immutable, partitioned collection of elements that can be operated on in parallel. This class contains the basic operations available on all RDDs, such as map, filter, and persist. In addition, org.apache.spark.rdd.PairRDDFunctions contains operations available only on RDDs of key-value pairs, such as groupByKey and join; org.apache.spark.rdd.DoubleRDDFunctions contains operations available only on RDDs of Doubles; and org.apache.spark.rdd.SequenceFileRDDFunctions contains operations available on RDDs that can be saved as SequenceFiles. All operations are automatically available on any RDD of the right type (e.g. RDD[(Int, Int)]) through implicit.
>
> Internally, each RDD is characterized by five main properties:
> 
> * A list of partitions
> * A function for computing each split
> * A list of dependencies on other RDDs
> * Optionally, a Partitioner for key-value RDDs (e.g. to say that the RDD is hash-partitioned)
> * Optionally, a list of preferred locations to compute each split on (e.g. block locations for an HDFS file)
> 
> All of the scheduling and execution in Spark is done based on these methods, allowing each RDD to implement its own way of computing itself. Indeed, users can implement custom RDDs (e.g. for reading data from a new storage system) by overriding these functions. Please refer to the Spark paper for more details on RDD internals.


### 0x0304 RelationalGroupedDataset

Spark官方文档：

1. [org.apache.spark.sql.RelationalGroupedDataset](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.sql.RelationalGroupedDataset)
2. 

Class说明：

> A set of methods for aggregations on a DataFrame, created by groupBy, cube or rollup (and also pivot).
> 
> The main method is the agg function, which has multiple variants. This class also contains some first-order statistics such as mean, sum for convenience.


API:

> * def agg(expr: Column, exprs: Column*): DataFrame
>> Compute aggregates by specifying a series of aggregate columns.
>
> * def agg(exprs: Map[String, String]): DataFrame
>> (Java-specific) Compute aggregates by specifying a map from column name to aggregate methods.
>
> * def agg(exprs: Map[String, String]): DataFrame
>> (Scala-specific) Compute aggregates by specifying a map from column name to aggregate methods.
>
> * def agg(aggExpr: (String, String), aggExprs: (String, String)*): DataFrame
>> (Scala-specific) Compute aggregates by specifying the column names and aggregate methods.
>
> * def avg(colNames: String*): DataFrame
>> Compute the mean value for each numeric columns for each group.
>
> * def count(): DataFrame
>> Count the number of rows for each group.
>
> * def max(colNames: String*): DataFrame
>> Compute the max value for each numeric columns for each group.
>
> * def mean(colNames: String*): DataFrame
>> Compute the average value for each numeric columns for each group.
>
> * def min(colNames: String*): DataFrame
>> Compute the min value for each numeric column for each group.
>
> * def pivot(pivotColumn: Column, values: List[Any]): RelationalGroupedDataset
>> (Java-specific) Pivots a column of the current DataFrame and performs the specified aggregation.
>
> * def pivot(pivotColumn: Column, values: Seq[Any]): RelationalGroupedDataset
Pivots a column of the current DataFrame and performs the specified aggregation.
>
> * def pivot(pivotColumn: Column): RelationalGroupedDataset
>> Pivots a column of the current DataFrame and performs the specified aggregation.
>
> * def pivot(pivotColumn: String, values: List[Any]): RelationalGroupedDataset
Permalink
>> (Java-specific) Pivots a column of the current DataFrame and performs the specified aggregation.
> 
> * def pivot(pivotColumn: String, values: Seq[Any]): RelationalGroupedDataset
>> Pivots a column of the current DataFrame and performs the specified aggregation.
> 
> * def pivot(pivotColumn: String): RelationalGroupedDataset
>> Pivots a column of the current DataFrame and performs the specified aggregation.
>
> * def sum(colNames: String*): DataFrame
>> Compute the sum for each numeric columns for each group.
> 
> * def toString(): String


## Reference

1. [Scala 教程](https://www.runoob.com/scala/scala-tutorial.html)
2. [Spark SQL Guide](http://spark.apache.org/docs/2.4.4/sql-programming-guide.html)
3. [classDataset\[T\] extends Serializable](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.sql.Dataset)
4. [classSparkConf extends Cloneable with Logging with Serializable](http://spark.apache.org/docs/2.4.4/api/scala/index.html#org.apache.spark.SparkConf)
5. [Hive Schema version 2.1.0 does not match metastore(版本不匹配）解决](https://blog.csdn.net/qq_27882063/article/details/79886935)
6. []()
