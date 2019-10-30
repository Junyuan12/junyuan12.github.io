---
title: 分类损失用交叉熵的原因
date: 2019-10-11 09:19:33
categories: 深度学习
tags:
     - 深度学习
mathjax: true
---

最近一直在纠结的一个问题，为什么分类损失用交叉熵（cross entry）而不用看起来更简单的二次代价函数（square mean），偶然间搜到一篇博客（[查看原文](<https://blog.csdn.net/zlsjsj/article/details/80264927>)），貌似是领悟了一点，简单整理下。

<!-- more -->

### 前向传播

前向传播的流程都是一样的，以二分类为例，以 $Sigmoid$ 为激活函数，当到达网络最后一层时：

$z = w \cdot x + b$

$a = \sigma (z) = sigmoid(z)$

### 反向传播

为了方便计算，只取一个样本：

#### 1. 二次代价函数

损失：
$$
\begin{equation}\begin{split} 
c = \frac{(a-y)^2}{2}
\end{split}\end{equation}
$$


$w$ 和 $b$ 的梯度为：
$$
\begin{equation}\begin{split} 
\frac{\partial c}{\partial w} &= (a - y ) \cdot a' \\
&= (a - y) \cdot [\sigma(wx + b)]' \\
&= (a - y) \cdot \sigma'(z) \cdot x
\end{split}\end{equation}
$$

$$
\begin{equation}\begin{split} 
\frac{\partial c}{\partial b} &= (a - y ) \cdot a' \\
&= (a - y) \cdot [\sigma(wx + b)]' \\
&= (a - y) \cdot \sigma'(z)
\end{split}\end{equation}
$$

#### 2. 交叉熵

损失：
$$
\begin{equation}\begin{split} 
c = -y log(a) - (1 - y)log(1 - a)
\end{split}\end{equation}
$$

$w$ 和 $b$ 的梯度为：
$$
\begin{equation}\begin{split} 
\frac{\partial c}{\partial w} &= -(\frac{y}{a} - \frac{1 - y}{1 - a}) \frac{\partial \sigma(z)}{\partial w} \\ 
&= -(\frac{y}{a} - \frac{1 - y}{1 - a}) \cdot \sigma'(z) \cdot x \\
&= -( \frac{y}{\sigma(z)} - \frac{1 - y}{1 - \sigma(z)} ) \cdot \sigma(z) \cdot (1 - \sigma(z)) \cdot x \\
&= -( \frac{(1 - \sigma(z))y - \sigma(z)(1 - y)}{\sigma(z)(1 - \sigma(z)} ) \cdot \sigma(z) \cdot (1 - \sigma(z)) \cdot x \\
&= (\sigma(z) - y) \cdot x
\end{split}\end{equation}
$$

$$
\begin{equation}\begin{split} 
\frac{\partial c}{\partial b} = \sigma(z) - y
\end{split}\end{equation}
$$

**Note: ** Sigmoid的导数为 $\sigma'(z) = \sigma(z) \cdot (1 - \sigma(z)$

### 结论

在反向传播时：

1. 对于square mean在更新 $w$，$b$ 时候， $w$，$b$ 的梯度跟激活函数的梯度成正比，激活函数梯度越大， $w$，$b$ 调整就越快，训练收敛就越快，但是Simoid函数在值非常高时候，梯度是很小的，比较平缓。

2. 对于cross entropy在更新 $w$，$b$ 时候， $w$，$b$ 的梯度跟激活函数的梯度没有关系了，$\sigma'(z)$ 已经被抵消掉了，其中$(\sigma(z) - y)$ 表示的是预测值跟实际值差距，如果差距越大，那么 $w$，$b$ 调整就越快，收敛就越快。