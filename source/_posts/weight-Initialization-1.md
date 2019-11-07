---
title: Weight Initialization in Neural Networks$:$ A Journey From the Basics to Kaiming(上)
date: 2019-10-17 11:11:11
categories: 深度学习
tags:
     - 深度学习
mathjax: true
---

> 或许去趟沙滩 或许去看看夕阳
>
> 或许任何一个可以想心事的地方
>
> ​													——《手写的从前》

<!-- more -->

原文链接[点击前往]( https://towardsdatascience.com/weight-initialization-in-neural-networks-a-journey-from-the-basics-to-kaiming-954fb9b47c79 )。

本文通过简短的实验说明为什么适当的初始化权值在深度神经网络训练中如此重要。 分别用`Tensorflow2.0`和`Pytorch`实现。

### Why Initialize Weight

  权重初始化的目的是防止层激活输出在深度神经网络的正向传递过程中爆炸或消失。如果发生以上任何一种情况，损失梯度不是太大就是太小，无法有利地反向传播，如果发生了以上的情况，网络收敛则需要更长的时间。

 矩阵乘法是神经网络的基本数学运算。在多层的深度神经网络中，一个前向传播需要在每一层上执行入和权重矩阵之间的矩阵乘法，得到的结果又作为下一层的输入，以此类推。 

 举一个简单的例子，假设我们有一个向量 $x$ 作为网络的输入。训练神经网络时的标准做法是对输入做一个处理，使它的值落在均值为 $0$、标准差为 $1$ 的正态分布内。 

```python
# pytorch输入
th_x= torch.randn(512)
# tensorflow输入
tf_x = tf.random.normal([512, 1], mean=0.0, stddev=1.0)
```

```python
# 查看输入x的均值和标准差
th_mean = th_x.mean()
th_var = th_x.var()
print('mean of th_x is {:.5f}, var of th_x is {:.5f}'.format(th_mean, th_var))
# mean of th_x is -0.00323, var of th_x is 1.04362

tf_mean, tf_var = tf.nn.moments(tf_x, axes=0)
print('mean of tf_x is {:.5f}, var of tf_x is {:.5f}'.format(tf_mean[0], tf_var[0]))
```

 假设我们有一个简单的 $100$ 层网络，没有激活函数，每一层都有一个矩阵 $a$ 作为权值。为了完成单个的前向传递，我们必须在层输入和每一层的权值之间执行一个矩阵乘法，这将产生总计 $100$ 个连续的矩阵乘法。 

 由此看出使用相同的标准正态分布处理输入并不是一个好的办法。为了找出原因，我们可以模拟前向传播通过我们假设的网络。 

```python
# pytorch
for i in range(100):
    th_a = torch.randn(512, 512)
    th_x = th_a @ th_x
print(th_x.mean(), th_x.std())
# tensor(nan) tensor(nan)

th_x = torch.randn(512)

for i in range(100):
    th_a = torch.randn(512, 512)
    th_x = th_a @ th_x
    if torch.isnan(th_x.std()): break
i
# 28
```

```python
# tensorflow
for i in range(100):
    tf_a = tf.random.normal([512, 512])
    tf_x = tf.matmul(tf_a, tf_x)
tf_mean, tf_var = tf.nn.moments(tf_x,axes=0)
print(tf_mean, tf_var)
# tf.Tensor([nan], shape=(1,), dtype=float32) tf.Tensor([nan], shape=(1,), dtype=float32)

tf_x = tf.random.normal([512, 1])

for i in range(100):
    tf_a = tf.random.normal([512, 512])
    tf_x = tf.matmul(tf_a, tf_x)
    tf_mean, tf_var = tf.nn.moments(tf_x,axes=0)
    if np.isnan(tf_var): break # 在tf2.0中没有找到判断nan的函数，使用numpy判断
i
# 27
```

网络在 $29$ （tensorflow在 $28$ 层）层输出已经爆炸了。

除了防止输出结果爆炸，还要防止输出消失。调整网络的权值，使它们仍然落在均值为0的正态分布内，但标准差为0.01 。

```python
# pytorch
th_x = torch.randn(512)

for i in range(100):
    th_a = torch.randn(512, 512) * 0.01
    th_x = th_a @ th_x
th_x.mean(), th_x.var()
# (tensor(0.), tensor(0.))
```

```python
# tensorflow
tf_x = tf.random.normal([512, 1])

for i in range(100):
    tf_a = tf.random.normal([512, 512], mean=0.0, stddev=0.01)
    tf_x = tf.matmul(tf_a, tf_x)
    tf_mean, tf_var = tf.nn.moments(tf_x,axes=0)
tf_mean.numpy(), tf_var.numpy()
# (array([0.], dtype=float32), array([0.], dtype=float32))
```

 在上面假设的正向传播过程中，激活输出完全消失。

总而言之，当权重初始化过大或过小时，网络都不能很好的学习。 

### How can we find the sweet spot?

 如上所述，完成通过神经网络的前向传播所需要的数学只不过是一系列矩阵乘法。如果我们有一个输出 $y$ 是输入向量 $x$ 和权重矩阵 $a$ 之间的矩阵乘法的乘积， $y$ 中的每个元素 $i$ 被定义为

$$
\begin{equation}\begin{split} 
y_i = \sum_{k=1}^{n-1}{a_{i,k}x_k}
\end{split}\end{equation}
$$

 其中 $i$ 是给定的权矩阵 $a$ 的行索引，$k$ 是给定的权矩阵 $a$ 的列索引和输入向量 $x$ 的元素索引，$n$ 是 $x$ 中元素的范围或总数，这在Python中也可以定义为:  

```python
y[i] = sum([c*d for c,d in zip(a[i], x)])
```

 可以证明，在给定的层上，我们从标准正态分布初始化的输入 $x$ 和权重矩阵 $a$ 的矩阵乘积通常将非常接近输入层节点数的平方根， 它在我们的例子中是 $\sqrt512$。 

```python
# pytorch
th_mean, th_var = 0., 0.
for i in range(10000):
    th_x = torch.randn(512)
    th_a = torch.randn(512, 512)
    th_y = th_a @ th_x
    th_mean += th_y.mean().item()
    th_var += th_y.pow(2).mean().item()
print(th_mean/10000, np.sqrt(th_var/10000))
print(np.sqrt(512))
# -0.002851470077782869 22.624340364162048
# 22.627416997969522
```

```python
# tensorflow
tf_mean, tf_var = 0., 0.
for i in range(10000):
    tf_x = tf.random.normal([512, 1])
    tf_a = tf.random.normal([512, 512])
    tf_y = tf.matmul(tf_a, tf_x)
    tf_mean += tf.reduce_mean(tf_y)
    tf_var += tf.reduce_mean(tf.square(tf_y))
print(tf_mean/10000, np.sqrt(tf_var/10000))
print(np.sqrt(512))
# tf.Tensor(-0.011328138, shape=(), dtype=float32) 22.623604
# 22.627416997969522
```

 如果从定义矩阵乘法的角度来看，这个性质并不奇怪： 为了计算 $y$，我们将输入 $x$ 的一个元素与权重 $a$ 的一列相乘，得到 $512$ 个乘积。在我们的示例中，$x$ 和 $a$ 都使用标准正态分布初始化，这 $512$ 个乘积的均值为 $0$，标准差为 $1$。 

```python
# pytorch
th_mean, th_var = 0., 0.
for i in range(10000):
    th_x = torch.randn(1)
    th_a = torch.randn(1)
    th_y = th_a * th_x
    th_mean += th_y.item()
    th_var += th_y.pow(2).item()
print(th_mean/10000, np.sqrt(th_var/10000))
# 0.012777583549162991 0.9860565282847021
```

```python
# tensorflow
# tensorflow
tf_mean, tf_var = 0., 0.
for i in range(10000):
    tf_x = tf.random.normal([1])
    tf_a = tf.random.normal([1])
    tf_y = tf.multiply(tf_a, tf_x)
    tf_mean += tf.reduce_mean(tf_y)
    tf_var += tf.reduce_mean(tf.square(tf_y))
print(tf_mean/10000, np.sqrt(tf_var/10000))
# tf.Tensor(-0.007080218, shape=(), dtype=float32) 1.004455
```

 因此，这 $512$ 个乘积的和的均值为 $0$，方差为 $512$，因此标准差为 $\sqrt 512$。 

 这就是为什么在我们上面的例子中，我们看到我们的层输出在 $29$ 次连续的矩阵乘法后爆炸。在我们最基本的 $100$ 层网络架构中，我们希望每个层的输出的标准偏差为 $1$。可以想象，这将允许我们在任意多的网络层上重复矩阵乘法，而不会触发或消失。 

 如果我们首先通过将所有随机选择的值除以 $\sqrt 512$ 来对权重矩阵 $a$ 进行缩放，那么填充输出 $y$ 的一个元素的元素乘法现在的平均方差只有 $1/\sqrt 512$。 

```python
# pytorch
th_mean, th_var = 0., 0.
for i in range(10000):
    th_x = torch.randn(1)
    th_a = torch.randn(1) * np.sqrt(1./512)
    th_y = th_a * th_x
    th_mean += th_y.item()
    th_var += th_y.pow(2).item()
print(th_mean/10000, th_var/10000)
print(1/512)
# 0.0005586366911495901 0.0019827878041473895
# 0.001953125
```

```python
# tensorflow
tf_mean, tf_var = 0., 0.
for i in range(10000):
    tf_x = tf.random.normal([1])
    tf_a = tf.random.normal([1]) * np.sqrt(1./512)
    tf_y = tf.multiply(tf_a, tf_x)
    tf_mean += tf.reduce_mean(tf_y)
    tf_var += tf.reduce_mean(tf.square(tf_y))
print(tf_mean/10000, tf_var/10000)
print(1/512)
# tf.Tensor(-0.00051529706, shape=(), dtype=float32) tf.Tensor(0.002007504, shape=(), dtype=float32)
# 0.001953125
```

 这意味着矩阵  $y$ 的标准差为 $1$，其中包含输入 $x$ 与权重 $a$ 相乘生成的 $512$ 个值中的每一个。让我们通过实验来证实这一点。 

```python
# pytorch
th_mean, th_var = 0., 0.
for i in range(10000):
    th_x = torch.randn(512)
    th_a = torch.randn(512, 512) * np.sqrt(1./512)
    th_y = th_a @ th_x
    th_mean += th_y.mean().item()
    th_var += th_y.pow(2).mean().item()
print(th_mean/10000, np.sqrt(th_var/10000))
# 0.0008099440926453099 1.0001812369800038
```

```python
# tensorflow
tf_mean, tf_var = 0., 0.
for i in range(10000):
    tf_x = tf.random.normal([512, 1])
    tf_a = tf.random.normal([512, 512]) * np.sqrt(1./512)
    tf_y = tf.matmul(tf_a, tf_x)
    tf_mean += tf.reduce_mean(tf_y)
    tf_var += tf.reduce_mean(tf.square(tf_y))
print(tf_mean/10000, np.sqrt(tf_var/10000))
# tf.Tensor(0.00010935542, shape=(), dtype=float32) 1.0003854
```

 现在让我们重新运行我们的100层网络。和之前一样，我们首先从 $[-1,1]$ 内部的标准正态分布中随机选择层权值，但这次我们将这些权值缩放 $1/\sqrt n$*，其中* $n$ 是一层的网络输入连接数，在我们的示例中为 $512$。 

```python
# pytorch
th_x = torch.randn(512)

for i in range(100):
    th_a = torch.randn(512, 512) * np.sqrt(1./512)
    th_x = th_a @ th_x
th_x.mean(), th_x.std()
# (tensor(-0.0199), tensor(1.1058))
```

```python
# tensorflow
tf_x = tf.random.normal([512, 1])

for i in range(100):
    tf_a = tf.random.normal([512, 512]) * np.sqrt(1./512)
    tf_x = tf.matmul(tf_a, tf_x)
tf_mean, tf_var = tf.nn.moments(tf_x,axes=0)
tf_mean.numpy(), tf_var.numpy()
# (array([-0.06913895], dtype=float32), array([0.9944094], dtype=float32))
```

 乍一看似乎时可以收工了，但现实世界的神经网络并不像第一个例子所显示的那么简单，为了简单起见，这里省略了激活函数。然而，在实际中，神经网络每一层的结尾都会加上一层激活函数，以此，深度神经网络可以近似的逼近一个复杂的函数来描述实际生活中的现象，比如手写数字的分类 。