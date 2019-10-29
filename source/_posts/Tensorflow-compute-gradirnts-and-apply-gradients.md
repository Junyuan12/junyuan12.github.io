---
title: Tensorflow compute_gradirnts和apply_gradients原理浅析
date: 2019-10-29 11:03:37
categories: 
- 深度学习
tags: 
- tensorflow
---

转自[TensorFlow学习笔记之--[compute_gradients和apply_gradients原理浅析]]( https://www.cnblogs.com/marsggbo/p/10056057.html )

<!-- more -->

最近在看fatser rcnn的源码，发现优化梯度的时候不是直接用的`Optimizer.minimize()`，而是先计算梯度`Optimizer.compute_gradients()`，然后`Optimizer.apply_gradients()`。

### optimizer.minimizer(loss, var_list)

 我们都知道，TensorFlow为我们提供了丰富的优化函数，例如GradientDescentOptimizer。这个方法会自动根据loss计算对应variable的导数。示例如下： 

```python
loss = ...
opt = tf.tf.train.GradientDescentOptimizer(learning_rate=0.1)
train_op = opt.minimize(loss)
init = tf.initialize_all_variables()

with tf.Seesion() as sess:
    sess.run(init)
    for step in range(10):  
      session.run(train_op)
```

 首先我们看一下`minimize()`的源代码(为方便说明，部分参数已删除): 

```python
def minimize(self, loss, global_step=None, var_list=None, name=None):

    grads_and_vars = self.compute_gradients(loss, var_list=var_list)

    vars_with_grad = [v for g, v in grads_and_vars if g is not None]
    if not vars_with_grad:
      raise ValueError(
          "No gradients provided for any variable, check your graph for ops"
          " that do not support gradients, between variables %s and loss %s." %
          ([str(v) for _, v in grads_and_vars], loss))

    return self.apply_gradients(grads_and_vars, global_step=global_step,
                                name=name)
```

 由源代码可以知道`minimize()`实际上包含了两个步骤，即`compute_gradients`和`apply_gradients`，前者用于计算梯度，后者用于使用计算得到的梯度来更新对应的variable。下面对这两个函数做具体介绍。 

### compute_gradients(loss, val_list)

参数含义:

- **loss**: 需要被优化的Tensor
- **val_list**: Optional list or tuple of `tf.Variable` to update to minimize `loss`. Defaults to the list of variables collected in the graph under the key `GraphKeys.TRAINABLE_VARIABLES`.

简单说该函数就是用于计算loss对于指定val_list的导数的，最终返回的是元组列表，即[(gradient, variable),...]。

看下面的示例:

```python
x = tf.Variable(initial_value=50., dtype='float32')
w = tf.Variable(initial_value=10., dtype='float32')
y = w*x

opt = tf.train.GradientDescentOptimizer(0.1)
grad = opt.compute_gradients(y, [w,x])
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    print(sess.run(grad))
```

 返回值如下: 

```python
>>> [(50.0, 10.0), (10.0, 50.0)]
```

可以看到返回了一个list，list中的元素是元组。第一个元组第一个元素是50，表示∂y∂w∂y∂w的计算结果，第二个元素表示ww。第二个元组同理不做赘述。

其中`tf.gradients(loss, tf.variables)`的作用和这个函数类似,但是它只会返回计算得到的梯度，而不会返回对应的variable。

```python
with tf.Graph().as_default():
    x = tf.Variable(initial_value=3., dtype='float32')
    w = tf.Variable(initial_value=4., dtype='float32')
    y = w*x

    grads = tf.gradients(y, [w])
    print(grads)
    
    opt = tf.train.GradientDescentOptimizer(0.1)
    grads_vals = opt.compute_gradients(y, [w])
    print(grad_vals)

>>>
[<tf.Tensor 'gradients/mul_grad/Mul:0' shape=() dtype=float32>]
[(<tf.Tensor 'gradients_1/mul_grad/tuple/control_dependency:0' shape=() dtype=float32>, <tf.Variable 'Variable_1:0' shape=() dtype=float32_ref>)]
```

### apply_gradients(grads_and_vars,global_tep=None,name=None)

该函数的作用是将`compute_gradients()`返回的值作为输入参数对variable进行更新。

那为什么`minimize()`会分开两个步骤呢？原因是因为在某些情况下我们需要对梯度做一定的修正，例如为了防止梯度消失(gradient vanishing)或者梯度爆炸(gradient explosion)，我们需要事先干预一下以免程序出现**Nan**的尴尬情况；有的时候也许我们需要给计算得到的梯度乘以一个权重或者其他乱七八糟的原因，所以才分开了两个步骤。

### Example

 下面给出一个使用`tf.clip_by_norm`来修正梯度的例子: 

```python
with tf.Graph().as_default():
    x = tf.Variable(initial_value=3., dtype='float32')
    w = tf.Variable(initial_value=4., dtype='float32')
    y = w*x
    
    opt = tf.train.GradientDescentOptimizer(0.1)
    grads_vals = opt.compute_gradients(y, [w])
    for i, (g, v) in enumerate(grads_vals):
        if g is not None:
            grads_vals[i] = (tf.clip_by_norm(g, 5), v)  # clip gradients
    train_op = opt.apply_gradients(grads_vals)
    with tf.Session() as sess:
        sess.run(tf.global_variables_initializer())
        print(sess.run(grads_vals))
        print(sess.run([x,w,y]))
        
>>>     
[(3.0, 4.0)]
[3.0, 4.0, 12.0] 
```

### 参考

1. [TensorFlow中Optimizer.minimize()与Optimizer.compute_gradients()和Optimizer.apply_gradients()的用法]( https://blog.csdn.net/Huang_Fj/article/details/102688509 )
2. [TensorFlow2.0笔记12：梯度下降,函数优化实战,手写数字问题实战以及Tensorboard可视化！]( https://blog.csdn.net/abc13526222160/article/details/89497497#_3 )

