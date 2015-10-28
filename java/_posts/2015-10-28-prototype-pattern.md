---
layout: post
title: "[设计模式]原型模式-Prototype Pattern"
modified: 2015-10-28 20:14:44
excerpt: "设计模式之原型模式学习笔记"
tags: [设计模式, design patterns]
published: false
---

##1.定义
Specify the kinds of objects to create using a prototypical instance,and create new objects by copying this prototype.

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

![通用类图](https://raw.githubusercontent.com/chiemy/JavaDesignPatterns/master/PrototypePattern/prototype01.png)

Java原型模式的核心就是实现Cloneable接口然后覆写clone()方法，实现对对象的复用。

> 注：Cloneable只起到了标示作用，是一个空接口。clone方法是Object的方法。

##2.优点

- 性能优良

原型模式是内存二进制流的拷贝，要比直接new一个对象性能好的多，特别是在一个for循环内产生大量的对象时。

- 逃避构造函数的约束

这是它的优点也是缺点，直接在内存中拷贝，是不会执行构造函数的。

##4.使用场景

- 资源优化场景

类的初始化需要消耗很多资源的时候

- 性能和安全要求的场景

new一个对象需要非常繁琐的数据准备或访问权限时。

- 一个对象多个修改者的场景

一个对象需要提供给其他对象访问，而且各个调用者都需要修改其值时，可以考虑用原型模式拷贝多个对象供调用者使用

##5.注意事项

###5.1 构造函数不会执行
一个实现了Cloneable接口并实现clone方法的类A，有一个无参或有参构造B，通过new产生一个对象S，然后通过S.clone()方法产生了一个新的对象T，那么在对象拷贝时B是不会执行的。

###5.2 深拷贝和浅拷贝

####5.2.1 浅拷贝
Object类提供的clone方法只是拷贝本对象，其对象内部的数组、引用对象都不拷贝，还是指向原生对象的内部元素地址，而其他原始数据类型以及String类型都会被拷贝。

因此，浅拷贝情况下，原对象和拷贝对象对其内部的数组及引用对象进行修改时，是相互影响的，因为指向的是同一块内存地址。

####5.2.2 深拷贝
深拷贝能够使得原对象和拷贝对象对数组和引用对象的修改互不影响，实现方式就是在clone方法中，实现对除基本数据类型和String类型之外数据的拷贝。

###5.3 final的数据不能clone

一个声明为final的数据不能调用clone方法，否则编译报错。

##6.示例代码

Github:[Proxy Pattern]()

参考：

- 《设计模式之禅》
