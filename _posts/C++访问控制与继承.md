---
title: C++访问控制与继承
tags:
  - class
  - C++
  - 访问控制
  - 继承
permalink: c-aces-control-and-inheritance
date: 2015-12-25 21:56:04
---

## 类的三种用户

* 普通用户：该类用户编写代码使用类的对象，这部分代码只能访问类的**public**成员

* 类的实现者：该类用户负责编写类的成员和友元的代码，成员和友元能访问所有成员

* 派生类：类可以作为基类提供给它的派生类使用，派生类的实现者（**友元和成员**）能访问基类的protected成员，但是派生类的普通用户和派生类不能访问。换句话说就是在派生类的三种用户里面**只有派生类的实现者能访问基类的protected成员**。当然，派生类还能访问基类的public成员而不能访问基类的private成员

```C++
class Base
{
public:
    int pub_mem;
    int visit_pri(){ return pri_mem; } //实现者可以访问任意成员
protected:
    int pro_mem;
private:
    int pri_mem;
};

class Pub_Derv : public Base
{
public:
    //派生类作为基类的使用者可以通过以下两种方式访问基类的成员
    int visit_base_pub(){return pub_mem;}//实现者可以使用基类的public成员
    int visit_base_pro(){return pro_mem;}//实现者可以使用基类的protected成员
    
    int visit_base_pri(){return pri_mem;}//错误，private成员不能被派生类使用 
};

void visit_class_mem()
{
    Base b;
    int a = b.pri_mem;//错误，普通用户不能访问类的非public成员
    int a = b.pro_mem;//错误，原因同上
} 
```
可以看出三种用户中类的实现者级别最高，可以访问本类的所有成员和基类的protected成员；派生类的级别次之，只能访问它的基类的public和protected成员；而普通用户的级别最低，只能访问类的public成员。

## 决定某个类对继承而来的成员的访问权限的因素
 * 基类中该成员的访问说明符
 * 派生列表中的访问说明符
