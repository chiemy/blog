---
layout: post
title: "[设计模式]模板方法模式Template Method Pattern"
modified: 2015-10-22 16:17:40
excerpt: "设计模式之模板方法模式"
tags: [设计模式, design patterns]
published: true
---

##1.定义
Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Template method Lets subclasses redefine certain steps of an algorithm without changing the algorithm structrue.

在某个操作中定义一个逻辑的框架，将一些步骤延迟到子类里。使得子类在不改变算法结构的情况下可以重新定义该算法的某些步骤。

![通用类图](https://github.com/chiemy/JavaDesignPatterns/blob/master/TemplateMethodPattern/%E9%80%9A%E7%94%A8%E7%B1%BB%E5%9B%BE.png?raw=true)

AbstractClass叫抽象模板，它的方法分为两类：

- 基本方法

也叫基本操作，由子类实现的方法，并且在模板方法中被调用。

> 基本方法尽量设计为protected类型（迪米特法则）

- 模板方法

可以有一个或者几个，一般是一个具体的方法，也就是一个框架，实现对基本方法的调度，完成固定的逻辑。

> 一般被final修饰，防止恶意篡改。

- 钩子方法


##2.优缺点

###（1）优点

- 封装不变部分，扩展可变部分
- 代码复用，消除重复代码，提取公共部分代码，便于维护
- 行为由父类控制，子类实现

###（2）缺点
需要为每一个基本方法的不同实现提供一个子类，如果父类中可变的基本方法太多，将会导致类的个数增加，系统更加庞大，设计也更加抽象，此时，可结合桥接模式来进行设计。

##3.使用场景

- 多个子类有公有的方法，并且逻辑基本相同
- 重要、复杂的算法，可以把核心算法设计为模板方法，周边的相关细节功能则由各个子类实现。
- 重构时，把相同的代码抽取到父类中，然后通过钩子函数约束其行为。

##4.扩展，钩子方法的运用
钩子方法的引入使得子类可以控制父类的行为。钩子方法是被模板方法调用的，从而起到影响到模板方法的运行结果。

Example:

	public class Algorithm {
		public void templateMethod() {
				:
				.
			hookMethod();
				.
				:
		}

		public void hookMethod() {
			// default implementation
		}
	}

	public class RefinedAlgorithm extends Algorithm {
		public void hookMethod() {
			// refined implementation
		}
	}


##5.示例代码

Github:[TemplateMethodPattern](https://github.com/chiemy/JavaDesignPatterns/tree/master/TemplateMethodPattern)

参考：

- 《设计模式之禅》
- [模板方法模式深度解析（一)](http://blog.csdn.net/lovelion/article/details/8299794)
