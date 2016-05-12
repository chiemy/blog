---
layout: post
title: "[设计模式]代理模式-Proxy Pattern"
modified: 2015-10-27 10:49:13
excerpt: "设计模式之代理模式"
tags: [设计模式, design patterns]
published: true
---

##1.定义

Provide a surrogate or placeholder for another object to control access to it.

为对象提供一个代理，来控制对这个对象的访问。

![通用类图](https://raw.githubusercontent.com/chiemy/JavaDesignPatterns/master/ProxyPattern/proxy_pattern01.png)

类图中的三个角色：

- Subject-抽象主题角色

一个抽象类或者接口，无特殊要求。

- RealSubject-具体主题角色

被委托或被代理类，业务逻辑的真正执行者。

- Proxy-代理角色

委托类或代理类，在被代理类业务逻辑执行前后做预处理和善后工作。


##2.优点

- 职责清晰

真实的角色实现实际的业务逻辑，不用关心其他非本职的事务，通过代理完成一些事务。

- 高扩展性

如果业务逻辑发生变化，代理类无需做任何修改。

- 智能化

##4.扩展

###4.1普通代理

调用者只能访问代理角色，而不能访问真实角色，调用者必须知道代理的存在。

[示例代码](https://github.com/chiemy/JavaDesignPatterns/tree/master/ProxyPattern)

###4.2强制代理

调用者直接调用真实的角色，而不用关心代理是否存在，其代理的产生是由真实角色决定的。


[示例代码](https://github.com/chiemy/JavaDesignPatterns/tree/master/ProxyPattern)

##5.示例代码

Github:[Proxy Pattern](https://github.com/chiemy/JavaDesignPatterns/tree/master/ProxyPattern)

参考：

- 《设计模式之禅》
