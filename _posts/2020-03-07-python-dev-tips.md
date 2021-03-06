---
layout:     post
title:     Python dev tips
author:   风止
category: blog
description: 记录Python开发中遇到的小技巧or小技术等。
---

## 0x00 Introduction

在项目研发中，常常需要使用Python进行方法or算法验证，因此需要对编写Python脚本中遇到的问题以及解决办法做一个记录，便于后续查询。

## 0x01 Tips

### 0x0100 文件路径拆分

**API:**

* os.path.splitext():分割文件名和扩展名，return元组。
* os.path.split():分割文件路径和文件名，return元组。

**Test code:**

```python
# OS:macOS
filepath = "Test/test.py"
(filedir, filename) = os.path.split(filepath)
print(filename)
(filestr, fileext) = os.path.splitext(filename)
print(fileext)
```

**Output:**

```bash
test.py
.py
```

### 0x0101 文件路径拼接

**API:**

* os.path.join:拼接文件目录和文件名称，return文件路径。

**Test code:**

```python
dirpath = "Test"
file_name = "test1.txt"
filepath = os.path.join(dirpath, file_name)
print(filepath)
```

**Output:**

```bash
Test/test1.txt
```

### 0x0102 字典key查询

Python中的dict，其存储方式为key-value存储，查找速度极快。在使用时，常常需要检测key是否在字典中。

检测方法：使用in来判断。

```bash
>>> 'Thomas' in d
False
```

### 0x0103 excel解析

使用第三库：`xlrd`

**API:**

* xlrd.open_workbook(()：根据excel文件路径，打开Excel工作簿。
* sheet_by_index()：根据索引，获取工作簿的表格。注意：一个工作博可能有多个表格。
* col_values(): 根据表格列的索引值，获取表格列的值。
* row_values()：根据表格row index，获取表格行的值。

**Test Code：**

```python
excel_data = xlrd.open_workbook(excel_file)
# get col count of first table
col_count = excel_data.sheet_by_index(0).ncols
for i in range(0, col_count):
	#get col values for every col
	col_list = excel_data.sheet_by_index(0).col_values(i)
```

### 0x0104 time处理

使用time库

**API:**

* time.time()： 获取当前时间戳。
* time.localtime()：将时间戳转换为元组（struct_time）。
* time.strftime()：将元组时间（struct_time）转换为格式化时间字符串。
* time.strptime()：将格式化的时间字符串转换为元组时间（struct_time）。
* time.mktime()：将元组时间（struct_time）转换为时间戳。


**Test Code:** 

```python
>>print(time.time())

1587713833.201077

>>print(time.localtime(time.time()))

time.struct_time(tm_year=2020, tm_mon=4, tm_mday=24, tm_hour=15, tm_min=45, tm_sec=57, tm_wday=4, tm_yday=115, tm_isdst=0)

>>print(time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(time.time())))

2020-04-24 15:37:13

>> str_time = '2019-02-26 13:04:41'
>> print(time.strptime(str_time, '%Y-%m-%d %H:%M:%S'))

time.struct_time(tm_year=2019, tm_mon=2, tm_mday=26, tm_hour=13, tm_min=4, tm_sec=41, tm_wday=1, tm_yday=57, tm_isdst=-1)

>> print(time.mktime(time.strptime(str_time, '%Y-%m-%d %H:%M:%S')))

1551157481.0
```

计算两个时间点之间的间隔时间，测试代码：

```python
time1 = datetime.datetime(2020, 6, 28, 12, 30, 55)
time2 = datetime.datetime(2020, 7, 1, 12, 31, 55)
time_range = time2 - time1
# 提取两个时间点之间的days
print("days: {0}".format(time_range.days))
# 提取两个时间点之间的seconds，除去days
print("seconds {0}".format(time_range.seconds))
# 提取两个时间点之间的total seconds
print("total seconds: {0}".format(time_range.total_seconds()))
```
 
 Output:
 
```bash
<class 'datetime.timedelta'>
days: 3
seconds 60
total seconds: 259260.0
```

注意⚠️：两个时间点相减之后的类型是`datetime.timedelta`。


### 0x0105 json数据处理

待补充

### 0x0106 检测元素的类型

```python
    test_list = [1, 2]
    print(isinstance(test_list, list))
    print(isinstance(22, float))
```

Output:

```bash
True
False
```

## 0x02 Python Release

参考[9]，打包python程序为tar或者zip包。



## Reference

1. [Python 日期和时间戳的转换](https://www.cnblogs.com/strivepy/p/10436213.html)
2. [Python如何处理Excel表格？良心推荐！](https://www.jianshu.com/p/ae01855198fb)
3. [python路径拼接os.path.join()函数的用法](https://www.cnblogs.com/an-ning0920/p/10037790.html)
4. [使用dict和set](https://www.liaoxuefeng.com/wiki/1016959663602400/1017104324028448)
5. [Python获取文件路径、文件名和扩展名](https://blog.csdn.net/lilongsy/article/details/99853925)
6. [常用模块之datetime模块（date，time，datetime，timedelta）](https://blog.csdn.net/z_xiaochuan/article/details/81324367)
7. [python 日期、时间、字符串相互转换](https://www.cnblogs.com/huhu-xiaomaomi/p/10338472.html)
8. [python判断元素是什么类型](https://blog.csdn.net/m0_37490554/article/details/104795514)
9. [python模块的打包setuptools](https://www.cnblogs.com/skying555/p/5191503.html)
10. 