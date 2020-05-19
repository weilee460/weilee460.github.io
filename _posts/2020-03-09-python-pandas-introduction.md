
---
layout:     post
title:    Pandas Simple Use
category: blog
description: ，由于工作中常常需要处理大量的数据，而使用Pandas来处理，效率较高，因此需要熟悉一下这个强大的工具。以官方提供的《十分钟入门》文档为参考。
---


## 0x00 Introduction

工作中有很多数据需要处理，使用csv模块读取数据，使用Python List来处理数据，处理速度较慢。因此考虑使用第三方库“Pandas”来处理数据。结果表明，Pandas处理数据的速度远远快于个人编写的Python List处理程序。因此这里记录一下主要的用法。

## 0x01 Data Type

Pandas有两种数据类型：Series和DataFrame。

**Series** 官方文档描述：

> Series 是带标签的一维数组，可存储整数、浮点数、字符串、Python 对象等类型的数据。轴标签统称为索引。调用 pd.Series 函数即可创建 Series：

```python
s = pd.Series(data, index=index)
```

**DataFrame** 官方文档描述：

> DataFrame 是由多种类型的列构成的二维标签数据结构，类似于 Excel 、SQL 表，或 Series 对象构成的字典。DataFrame 是最常用的 Pandas 对象，与 Series 一样，DataFrame 支持多种类型的输入数据：
> 
> * 一维 ndarray、列表、字典、Series 字典
> * 二维 numpy.ndarray
> * 结构多维数组或记录多维数组
> * Series
> * DataFrame

总结起来，DataFrame就是类似于数据库中的表结构数据。

## 0x02 Data Operation

按行取数据：

```python
# 选取单行
df.iloc[row_index]

# 选取前三行数据
df[0:3]
```

按列取值：

```python
# 选取单列数据
df['col_name']

# 选取多列数据
df.loc[:, ['col1_name', 'col2_name']]
```

按条件选取：

```python
# 选取A列中值大于0的行
df[df.A > 0]
```

按列的值大小排序：

```python
df.sort_values(by='col_name')
```

读取csv文件数据：

```python
pd.read_csv(csvfile_path)
```

将数据写入csv文件：

```python
df.to_csv(csvfile_path)
```

## 0x03 Example

**创建DataFrame**：

```python
	import pandas as pd
	
	
    test_data = [
        ["2020-05-20 10:03:56", "192.168.1.2", "0875"],
        ["2020-05-19 12:03:56", "192.168.1.4", "0877"],
        ["2020-05-20 12:13:56", "192.168.1.2", "0867"],
        ["2020-05-20 10:03:55", "192.168.1.7", "0987"],
        ["2020-05-10 12:23:56", "192.168.1.4", "0775"],
        ["2020-05-12 18:05:06", "192.168.1.2", "0875"],
        ["2020-05-15 10:43:55", "192.168.1.7", "0587"]
    ]
    test_datadf = pd.DataFrame(test_data)
    print(test_datadf)
```

Output:

```bash
                     0            1      2
0  2020-05-20 10:03:56  192.168.1.2  0875
1  2020-05-19 12:03:56  192.168.1.4  0877
2  2020-05-20 12:13:56  192.168.1.2  0867
3  2020-05-20 10:03:55  192.168.1.7  0987
4  2020-05-10 12:23:56  192.168.1.4  0775
5  2020-05-12 18:05:06  192.168.1.2  0875
6  2020-05-15 10:43:55  192.168.1.7  0587
```

注：上述的DataFrame创建表明，其创建中使用的嵌套list时，可理解为以**行**来处理。这样保证了嵌套list的原来的语义。

**修改列名称，表明实际的含义**。添加代码如下：

```python
test_datadf.rename(columns={0: 'info_time', 1: 'ip', 2: 'id'}, inplace=True)
```

Output:

```bash
             info_time       ip       id
0  2020-05-20 10:03:56  192.168.3.2    0875
1  2020-05-19 12:03:56  192.168.3.4    0877
2  2020-05-20 12:13:56  192.168.3.2    0867
3  2020-05-20 10:03:55  192.168.3.7    0987
4  2020-05-10 12:23:56  192.168.3.4    0775
5  2020-05-12 18:05:06  192.168.3.2    0875
6  2020-05-15 10:43:55  192.168.3.7    0587
```

**获取上述数据中蕴含的不同IP列表**。添加代码如下：

```python
dst_ip_list = test_datadf['ip'].unique()
```

Output:

```bash
['192.168.1.2' '192.168.1.4' '192.168.1.7']
```

**将不同IP的ID按照时间先后，组合成序列**。添加代码如下：

```python
    split_datadf = test_datadf.groupby(['ip'])
    for key,value in split_datadf:
        dst_ip_info = value.sort_values("info_time")
        tech_list = dst_ip_info['id'].to_list()
        tech_list.append(key)
        print(tech_list)
```

Output:

```bash
['0875', '0875', '0867', '192.168.3.2']
['0775', '0877', '192.168.3.4']
['0587', '0987', '192.168.3.7']
```

**保存为CSV文件：**

```python
test_datadf.to_csv(file_name, mode='a+', index=0)
```


## Reference

1. [十分钟入门 Pandas](https://www.pypandas.cn/docs/getting_started/10min.html)
2. [Pandas数据结构简介](https://www.pypandas.cn/docs/getting_started/dsintro.html)
3. []()