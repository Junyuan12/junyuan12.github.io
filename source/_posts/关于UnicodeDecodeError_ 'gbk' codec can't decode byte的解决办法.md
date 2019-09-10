---
title: UnicodeDecodeError_ 'gbk' codec can't decode byte的解决办法
date: 2019-03-27 12:00:00
categories: python
tags:
     - python
     - DecodeError
     - 机器学习
---

这个《机器学习实战》朴素贝叶斯算法中遇到的问题。

<!-- more -->

﻿#### 问题描述

**UnicodeDecodeError: 'gbk' codec can't decode byte 0xae in position 199: illegal multibyte sequence**

#### 解决方案
1. `\email\ham`中的`23.txt`中第二段多了一个®（使用记事本打开显示的是“？”），导致解码失败，删除‘®’之后便可以继续执行。
2. 使用`windows-1252`编码的方式读取txt，open txt文档时加一条`encoding='windows-1252'`，
`open('email/spam/%d.txt' % i, encoding="windows-1252").read()`。

#### 问题分析
**UnicodeDecodeError**解释为Unicode的解码（decode）出现错误了，也就当前正在处理某种编码类型的字符串，是想要将该字符串去解码，变成Unicode，但是在解码的过程中发生错误了。
分别测试`\email\ham\`中的每一个txt文档，只有`23.txt`报错：
```python
    ...: file = open('email\\ham\\23.txt')
    ...: fileRead = file.read()
    ...: wordList = bayes.textParse(fileRead)
    ...: docList.append(wordList)
    ...: fullText.extend(wordList)
    ...: classList.append(1)
    ...:
---------------------------------------------------------------------------
UnicodeDecodeError                        Traceback (most recent call last)
<ipython-input-33-598ed4b80c9a> in <module>()
      1
      2 file = open('email\\ham\\23.txt')
----> 3 fileRead = file.read()
      4 wordList = bayes.textParse(fileRead)
      5 docList.append(wordList)

UnicodeDecodeError: 'gbk' codec can't decode byte 0xae in position 199: illegal
multibyte sequence
```
`23.txt`第二段（记事本打开“®”显示为“？”）：
SciFinance® is a derivatives pricing and risk model development tool that automatically generates C/C++ and GPU-enabled source code from concise, high-level model specifications. No parallel computing or CUDA programming expertise is required.
使用`gbk`无法解码“®”，在sublime text中查看`23.txt`的编码方式为`windows 1252`，所以使用`windows 1252`的编码方式打开txt文档，程序便能够正常执行了。
