---
title: Y组合子
permalink: also-talk-about-the-y-group-zygote
date: 2016-12-25 18:08:21
---

Y组合子要解决的问题是如何用纯正的lambda表达式实现递归
以阶乘为例，可以采用下面的代码以递归的形式表达：
```python
f = lambda n: n * f(n - 1) if n > 1 else 1
```
要求一个自然数n的阶乘只要调用`f(n)`即可
上述代码包含了一个赋值语句，而纯正的lambda表达式是没有赋值语句的，那么用纯正的lambda表达式能否实现递归呢？
这就是Y组合子要解决的问题，解设我们已经得到了一个Y组合子名字叫做Y，可以定义如下lambda表达式：
```python
g = lambda self: lambda n: n * self(n - 1) if n > 1 else 1
```

然后用Y包装一下g得到g_，即：
```python
g_ = Y(g)
```

要求一个自然数n的阶乘，调用`g_(n)`即可
也就是说，有了Y组合子，我们只需要比较简单地、**与用赋值语句差不多的方式** 定义一个lambda表达式，再用Y组合子包装一下，就可以实现与赋值语句定义相同的效果  

不难看出，我们可以用类似定义阶乘的方式定义出任何需要递归调用的表达式，方法就是在表达式最外层引入一个自由变量self来指代表达式本身，而在内部用self来代替自己以达到递归调用的效果

上述讨论的基本假设都是存在Y组合子，下面我们来试着求出Y组合子以证明Y组合子是存在的

以g为例，重点是指代g的那个self, 如何保证在调用`self(n-1)`的时候效果和调用`g_(n-1)`一样呢？  

g本身是不能接受一个n的，必须得指定一个self，调用`g(self)`之后才有可能再接受一个n，于是我们猜测Y包装g的时候构造了这样一个self并且返回的是和`g(self)`同阶的表达式  

同阶这个概念还是太过模糊，不妨假设`Y(g)=g(self)`  

再进一步，我们为了保证`self(n-1)`和`g_(n-1)`效果一样再假设`self=g_=Y(g)`  

这样，基于以上假设，只要`g(self)=self`就能保证`self(n-1)`的效果和`g_(n-1)`的效果是一样的  

虽然假设条件过于苛刻而失去了通用性，但是只要能作为充分条件求出一个Y组合子便能证明这样的Y组合子是存在的  

那么现在只要给出满足`g(self)=self`的self就可以构造出Y组合子了
于是self正是g的不动点，Y组合子又叫不动点组合子就是这么来的  

受知识所限，我也不知道应该怎样求lambda表达式的不动点，翻阅资料得知，self应该是`gen(gen)`这样的形式，即`self=gen(gen)`，其中
```python
gen = lambda x: g(x(x))
```
于是`gen(gen) = g(gen(gen))`
（关于gen的合理性由于笔者目前知识有限，先不作讨论，待填坑）

这样就找到了g的不动点  

现在我们开始构造Y组合子：
由上面的讨论可以知道`Y(g)=self=gen(gen)=g(gen(gen))`
则
```python
Y = lambda f: f(gen(gen))
```
即
```python
Y = lambda f: gen(gen)
```
即
```python
Y = lambda f: ((lambda x: f(x(x))) (lambda x: f(x(x))))
```
由此便得到了Y组合子

PS: 为方便理解以上lambda表达式全部都以Python语法表示，不保证一定能在Python解释器里执行
