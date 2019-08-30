---
title: Python 'Object of type '***' is not JSON serializable'
date: 2019-08-30 16:23:46
categories: python
tags:
     - python
     - json
---

这是想要将数据写到`json`文件遇到的问题，`Object of type int32/int64/ndarray is not JSON serializable`，归根结底都是一个问题。

<!-- more -->

#### 解决方案

由于`numpy.array`的数据类型不符合`json`的解码类型，使用`tolist()`方法将`array`转换成`list`。

**NOTE:** 不能使用`list()`方法，`list()`转换的list不会改变`array`的数据类型。

#### 问题分析

先看json可以解码的数据类型， Python操作json的[标准api库](<http://docs.python.org/library/json.html>)：

| JSON          | Python |
| :------------ | :----- |
| object        | dict   |
| array         | list   |
| string        | str    |
| number (int)  | int    |
| number (real) | float  |
| true          | True   |
| false         | False  |
| null          | None   |

支持的数据类型基本都是常见的`int`，`str`类型，而`numpy.array`的数据类型都是`numpy`内置的类型。

```python
import numpy as np
import json

y = []
for i in range(5):
    x = np.random.randint(10, size=(5,))
    y.append(x)
print(y)

d = {}
d["y"] = y
with open("res.json", "w") as f:
    json.dump(d, f)

Output:
[array([4, 6, 8, 8, 1]),
 array([2, 0, 9, 7, 1]),
 array([8, 7, 9, 7, 4]),
 array([2, 1, 2, 7, 2]),
 array([6, 4, 9, 3, 3])]

TypeError: Object of type 'ndarray' is not JSON serializable
```

`ndarray`类型不能保存，`list`总能保存吧，所以先尝试`list()`方法。

```python
y = []
for i in range(5):
    x = np.random.randint(10, size=(5,))
    y.append(list(x)
print(y)

Output:
[[7, 9, 6, 8, 1],
 [2, 8, 6, 7, 8],
 [8, 4, 2, 4, 3],
 [8, 0, 4, 2, 5],
 [7, 1, 4, 7, 0]]
```

看似没有任何问题，但是每次保存`json`都会遇到`Object of type 'int32' is not JSON serializable`的问题，参考json可以解码的数据类型，查看`x`中的数据类型。

```python
x = np.random.randint(10, size=(5,))
x = list(x)
print(type(x[0]))
Output:
<class 'numpy.int32'>
```

因此判断是`numpy.array`的数据类型的问题，所以尝试`tolist()`方法。

```python
x = np.random.randint(10, size=(5,))
x = x.tolist()
print(type(x[0]))
Output:
<class 'int'>
```

符合Json解码的标准，保存文件成功。

#### 问题

`tolist()`方法和`list()`方法的区别没查到，为什么一个改变数据类型，一个未改变数据类型。