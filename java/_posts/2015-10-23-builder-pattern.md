---
layout: post
title: "[设计模式]建造者模式-Builder Pattern"
modified: 2015-10-23 16:17:40
excerpt: "设计模式之建造者模式"
tags: [设计模式, design patterns]
published: true
---

## 1.定义
Separate the construction of a complex object from its representation so that the same construction process can create different representation.

将一个复杂对象的创建与它的表示分离，使得同样的构建过程可以创建不同的表示。

![](https://raw.githubusercontent.com/chiemy/JavaDesignPatterns/master/BuilderPattern/img1.png)

该模式中有四个角色：

- 产品类

通常实现模板方法模式

- 抽象建造者

规范产品的组建，一般是由子类实现。


- 具体建造者

实现抽象类定义的所有方法，并且返回一个组建好的对象。


- 导演类

负责安排已有模块的顺序，然后告诉建造者开始建造。


## 2.优点

- 封装性：使客户端不用知道产品内部组建的细节
- 建造者独立，容易扩展
- 便于控制细节风险：由于具体的建造者是独立的，因此可以对建造过程逐步细化，而不对其他的模块产生影响。


## 3.使用场景

- 相同的方法，不同的执行顺序产生不同的结果时
- 产品类非常复杂，或者产品类中调用顺序不同产生了不同的效能时
- 在系统创建过程中会使用到系统的一些其他对象，这些对象在产品对象的创建过程中不易得到时，作为补偿方法。

## 4.和工厂模式的不同
与建造者模式关注的是零件类型和装配顺序，而工厂方法的重点是创建，组装顺序不是它关心的。

## 5.示例代码

Github:[Builder Pattern](https://github.com/chiemy/JavaDesignPatterns/tree/master/BuilderPattern)

参考：

- 《设计模式之禅》
