---
title: python中super的使用
date: 2019-10-29 19:19:59
categories: python
tags:
     - python
---

> 等终等于等明等白

<!-- more -->

### 调用父类的初始化方法

继承父类后，直接调用父类的初始化方法。

定义一个人的类`Person`，包含人的名字和年龄，学生类`Student`继承自`Person`，多了一个分数变量：

```python
class Person():
    def __init__(self, name, age):
        self.name = name
        self.age = age
    def info(self):
        print("Person\'s name is {}, person\'s age is {}".format(self.name, self.age))

class Student(Person):
    def __init__(self, name, age, score):
        super().__init__(name, age)
        self.score = score
    def info(self):
        print("Student\'s name is {}, age is {}, score is {}".format(self.name, self.age, self.score))
        
p1 = Person("Zhangsan", 20)
p1.info()
>>> Person's name is zhangsan, person's age is 20
s1 = Student("Lisi", 18, 100)
s1.info()
>>> Student's name is Lisi, age is 18, score is 100
```

### 同时实现父类功能

如上一个示例，在`Student`类中重定义`info`方法直接就覆盖了父类`Person`中的`info`方法，如果需要同时实现父类的功能，就用`super`调用父类中的方法。只需要在`Student`中简单的修改下就可以：

```python
class Student(Person):
    def __init__(self, name, age, score):
        super().__init__(name, age)
        self.score = score
    def info(self):
        super().info()
        print("Student\'s score is {}".format(self.score))
        
s1 = Student("Lisi", 18, 100)
s1.info()
>>> Person's name is Lisi, person's age is 18
>>> Student's score is 100
```

使用`super`避免了直接用父类的名去调用，方便修改。 在上面的情况下，super 获得的类刚好是父类，但在其他情况就不一定了，**super 其实和父类没有实质性的关联**。 

### 多重继承

 上面说到的例子是单继承，用`父类名.属性`的方法调用出来代码维护时繁琐一点也并无不可，但多重继承时，还用这种方法来调用父类属性就会就回带来许多问题。假如有以下4个类，箭头指向父类，要在各子类方法中显示调用父类：

![Inheritance-relationship](Use-of-super-in-python\Inheritance-relationship.png)

```python
class Base():
    def fun(self):
        print("enter Base")
        print("leave Base")
        
class A(Base):
    def fun(self):
        print("enter A")
        Base.fun(self)
        print("leave A")
        
class B(Base):
    def fun(self):
        print("enter B")
        Base.fun(self)
        print("leave B")
        
class C(A, B):
    def fun(self):
        print("enter C")
        A.fun(self)
        B.fun(self)
        print("leave C")
        
c = C()
c.fun()

>>> enter C
>>> enter A
>>> enter Base
>>> leave Base
>>> leave A
>>> enter B
>>> enter Base
>>> leave Base
>>> leave B
>>> leave C
```

`Base`类被实例化了两次，发生菱形继承，出现调用不明确问题，并且会造成数据冗余的问题 。

#### MRO列表

 对于定义的每一个类，Python 会计算出一个**方法解析顺序（Method Resolution Order, MRO）列表**，**它代表了类继承的顺序**，类`C`的MRO表：

```python
C.mro()
>>> [__main__.C, __main__.A, __main__.B, __main__.Base, object]
```

  MRO 列表通过`C3线性化算法`来实现的，一个类的 MRO 列表就是合并所有父类的 MRO 列表，并遵循以下三条原则：

- 子类永远在父类前面
- 如果有多个父类，会根据它们在列表中的顺序被检查。
- 如果对下一个类存在两个合法的选择，选择第一个父类。

当用super调用父类的方法时，会按照mro表中的元素顺序去挨个查找方法。

```python
class Base():
    def fun(self):
        print("enter Base")
        print("leave Base")
        
class A(Base):
    def fun(self):
        print("enter A")
        super().fun()
        print("leave A")
        
class B(Base):
    def fun(self):
        print("enter B")
        super().fun()
        print("leave B")
        
class C(A, B):
    def fun(self):
        print("enter C")
        super().fun()
        print("leave C")
        
c = C()
c.fun()

>>> enter C
>>> enter A
>>> enter B
>>> enter Base
>>> leave Base
>>> leave B
>>> leave A
>>> leave C
```

为什么继承顺序是 $A \rightarrow B \rightarrow Base$ ?

#### super原理

super的工作原理如下：

```python
def super(cls, inst):
    mro = inst.__class__.mro()
    return mro[mro.index(cls) + 1]
```

其中，`cls`代表类，`inst`代表实例，上面的代码做了两件事：

- 获取`inst`的MRO列表
- 查找`cls`在当前MRO列表中的index, 并返回它的下一个类，即`mro[index + 1]`

当你使用 `super(cls, inst)` 时，Python 会在`inst`的 MRO 列表上搜索`cls`的下一个类。

所以根据MRO表，类`C`的下一个类是`A`，以此类推，得到 $A \rightarrow B \rightarrow Base$ 的继承顺序。

### 参考

1. [python 中 super函数的使用]( https://www.cnblogs.com/silencestorm/p/8404046.html )

2. [Python中super的用法]( https://blog.csdn.net/dxk_093812/article/details/87553937 )
3. [菱形继承问题和虚继承](https://www.cnblogs.com/lsh123/p/7912623.html)