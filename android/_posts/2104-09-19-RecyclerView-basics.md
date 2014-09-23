---
layout: post
type: android
title: "[译]RecyclerView基础"
modified: 2014-9-18 22:46:27
excerpt: "RecyclerView的使用方法"
tags: [android, RecyclerView ]
comments: true
---
[原博客地址：RecyclerView in Android: The basics](http://antonioleiva.com/recyclerview/)

新一届的Google I/O大会上公布了许多关于Android的消息。我们需要一段时间来消化Android L带来的新特性。

我研究了`RecyclerView`几天，接下来我会为大家展现过程中的一些发现。

##RecyclerView是什么？
`RecyclerView`是一个新的`ViewGroup`,通过简单的方法渲染那些以适配器为基础（adapter-based）的视图，它被认为是ListView和GridView的继承者，可以在最新的support-v7包中发现它。

`RecyclerView`保持了可扩展性，因此它可以创建任何你想到的布局，但会很困难（pain-in-the-ass），这就是Android，干什么都不容易。

如果要使用`RecyclerView`，那么你需要适应三个元素：

- RecyclerView.Adapter
- LayoutManager
- ItemAnimator

##RecyclerView.Adapter
`RecyclerView`包含了一个新的适配器，和你使用过的适配器类似，但还有一些新特性，例如一个必要的ViewHolder。你需要重写两个主要方法，一个是填充view和它的ViewHolder,另一个方法为view绑定数据。第一个方法只会在需要真正创建视图的时候才会被调用，不需要检查是否被回收。


<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=MyRecyclerAdapter.java"></script>


上边是一个简单的适配器的代码。但事情开始变得复杂起来了，我没发现`RecyclerView`有`OnItemClickListener`，所以这个适配器就应该处理这些事件了。

和`notifyDataSetChanged()`有些不同，如果你想要从适配器中移除成员，应该明确的告知适配器。


<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=RecyclerView_remove_item.java"></script>


##LayoutManager-布局管理器
这个类决定着视图在屏幕中所处的位置，但这是它众多智能之中的一个。




