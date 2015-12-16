---
layout: post
title: "[设计模式]命令模式-Command Pattern"
modified: 2015-12-09 16:11:54
excerpt: "设计模式之命令模式学习笔记"
tags: [设计模式, design patterns]
published: true
---

##1.定义
Encapsulate a request as an object,thereby letting you parameterize clients with different requests,queue or log requests,and support undoable operations.

将一个请求封装成一个对象，从而让你让你使用不同的请求把客户端参数化，对请求排队或者记录请求日志，可以提供命令的撤销和恢复功能。

<img src="https://raw.githubusercontent.com/chiemy/JavaDesignPatterns/master/CommandPattern/command_pattern_common.png"/>

- Receiver 接收者角色

执行具体任务的角色

- Command 命令角色

需要执行的命令都在这里声明

- Invoker 调用者角色

接收到命令并执行命令。


##2.优缺点

###2.1 优点
- 类间解耦

调用者角色与接受者角色之间没有任何的依赖，调用者实现功能时只需要调用Command抽象类的execute方法就可以，不需要了解到底是哪个接收者执行。

- 可扩展性

Command的子类可以非常容易的扩展，而调用者Invoker和高层次的模块Client不产生严重的代码耦合。

###2.2 缺点
类膨胀问题，有多少个命令就需要多少Command的子类。

##3.使用场景
只要认为是命令的地方就可以采用命令模式。


##4.示例代码

Github:[Command Pattern](https://github.com/chiemy/JavaDesignPatterns/tree/master/CommandPattern)

参考：
- 《设计模式之禅》