---
layout: post
title: "Gradle 中 implementation 与 compile 的区别"
modified: 2017-12-02 15:02:25
excerpt: "Gradle 3.0 后，依赖声明引入了 implementation 的方式，它与 compile 有什么不同？又有哪些优势？"
tags: [android, Gradle]
published: true
---

在 Android Studio 升级到 3.0 版本后，在我们创建新项目时我们发现了一些新的变化，`build.gradle` 使用了 `implementation` 来代替 `compile` 来添加依赖。那么 `implementation` 和 `compile` 有什么不同呢？

`implementation` 是在 Gradle 3.0 后引入的，在 3.0 之后 `compile` 已经被废弃，取而代之的是 `implementation` 和 `api` 。

## 作用

我们的项目依赖关系如下 app > module，我们在 module 的 `builde.gradle` 下通过不同的方式引入 `recyclerview` 的依赖，看有什么不一样的地方。

如果通过 `api` 或 `compile` 的方式，此时，我们在 app 中是可以使用 `RecyclerView` 类，我们可以称这种方式为”依赖的传递”；

如果通过 `implementation` 的方式，我们是不能使用 `RecyclerView` 类的。此依赖是不会暴露给 module 的使用者的，不存在“依赖的传递”。

优点：

- 没有依赖的传递，减少意外的使用（比如我们的类和依赖的类重名，很可能造成误用）
- 减少的**类路径大小（classpath size）**，编译更快
- 依赖发生变化，只有直接添加该依赖的模块会重新编译，而不会导致整个依赖结构都重新编译。如上例中：如果 recyclerview 依赖发生变化，只有 module 会重新编译，而 app 不会重新编译。

所以，我们应该尽量使用 `implementation` 来代替 `compile`。在有依赖传递的需求时，使用 `api`。


