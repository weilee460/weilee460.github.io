---
layout:     post
title:     Python dev tips
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


## Reference

1. []()
2. []()
3. [python路径拼接os.path.join()函数的用法](https://www.cnblogs.com/an-ning0920/p/10037790.html)
4. [使用dict和set](https://www.liaoxuefeng.com/wiki/1016959663602400/1017104324028448)
5. [Python获取文件路径、文件名和扩展名](https://blog.csdn.net/lilongsy/article/details/99853925)
6. 