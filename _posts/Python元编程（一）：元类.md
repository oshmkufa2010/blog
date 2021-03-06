---
title: Python元编程（一）：元类
tags:
  - Python
  - 元编程
permalink: 'python-(1):-metaclas-programing'
date: 2017-05-04 01:54:07
---
Python中类也是一个对象，类对象含有一个`__call__`方法，对一个类对象执行函数调用便会返回一个类的实例。如果我们定义一个类`A`:
```Python
class A(object):
    pass
```
此处的`A`便是一个对象，如果定义一个`A`的实例：
```Python
a = A()
```
便是调用了对象`A`的`__call__`方法。

那么类对象`A`是哪个类的实例呢？答案是元类(metaclass)，没错，元类的定义就是产生类的类，听起来有些绕。

Python默认的元类是`type`，也就是说，我们可以通过这样的方式来产生一个类：
```Python
A = type('A', (object,), dict(hello=lambda self: 'hello'))
```
`type`作为元类时需要三个参数，第一个参数是类的名字，第二个参数是父类的元组，第三个参数是类的成员方法组成的字典。

这里表示类对象`A`的名字是"A"，继承自`object`，定义了一个成员方法`hello`。

当`type`接受一个参数时，`type`会返回这个参数的类型；当`type`接受三个参数时，就会返回一个类对象。

这种方法以创建一个对象的方式定义了一个类，而不是用`class`关键字，这样就让Python的类型和对象一样可以被动态创建和使用，也就是说，类型不再像是C语言那样只在源代码里存在，而是和普通对象拥有一样的地位，可以在运行时存在，而且我们可以像对待普通对象那样对类型进行各种操作，这样就使得Python的抽象能力大大提高。(这也是我对元编程的理解)。

前面说过，`type`也是一个类，那我们自然也可以定义一个继承自`type`的类来扩充或者改变元类的功能，比如对`__call__`这种方法进行覆写。

比如我们要实现一个单例模式，可以这样写：
```Python
class SingletonMetaClass(type):
    def __init__(self, *args, **kwargs):
        super(SingletonMetaClass, self).__init__(*args, **kwargs)
        self.__ins = None

    def __call__(self, *args, **kwargs):
        if self.__ins is None:
            self.__ins = super(SingletonMetaClass, self).__call__(*args, **kwargs)
        return self.__ins

    def get_instance(self):
        return self()

Singleton = SingletonMetaClass('Singleton', (object,), dict())
```
这样就能保证每次调用`Singleton.get_instance()`得到的都是同一个对象。

注意每个方法中的`self`都是`SingletonMetaClass`的实例，也就是`Singleton`这个类对象，所以在`get_instance`方法中我们直接使用`self()`来创建`Singleton`的实例。

这种继承`type`类的写法和直接用`type`构建异曲同工，可以认为前者是利用了`class`这种语法糖来定义成员，而后者则通过构造函数来传递成员，所以后者更通用，更"meta"，而前者使用更方便。

此外，这里还有一种更不"meta"的写法：
```Python
class Singleton(object):
    # 这是Python2的写法，Python3直接在上面的括号里传metaclass参数
    __metaclass__ = SingletonMetaClass

    @classmethod
    def get_instance(cls):
        return cls()

```
其中`SingletonMetaClass`的定义同上，但是要去掉`get_instance`方法。

通过给一个类添加`__metaclass__`的方式，让这个类知道自己的元类，和通过调用元类的方法直接创建没有本质区别。

这种写法基本保持了`class`关键字定义类的方式，去掉`__metaclass__`就是一个普通类，加上就变成单例类，虽然失去一定灵活性，但是简洁明了，也使得`SingleMetaClass`更通用，是值得推荐的写法。
