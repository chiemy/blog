---
layout: post
type: android
title: "RecyclerView基础"
modified: 2014-9-24 17:10:55
excerpt: "RecyclerView的使用方法"
tags: [android, RecyclerView ,翻译]
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
这个类决定着视图在屏幕中所处的位置，但这是它众多职责之中的一个。它还负责管理滚动和回收。

`LayoutManager`只有一个实现类，叫做`LinearLayoutManager`。它有超过1500行的代码，可想有多复杂。此Manager可以模拟没有头部和页脚的`ListView`(横向或纵向)。

继承`LayoutManager`不适合新手，我们不得不一起努力来achieve`(取得；获得；实现；成功)``RecyclerView `的全部潜能。随着示例的不断深入，我会在较短的时间内上传一个`GridView`的实现。

这之后的关键点在于，从`LinearLayoutManager`继承并创建一个`BaseLayoutManager`。也许`support-v7`最终的版本会有更多更好的实现。

##ItemAnimator

`ItemAnimator`会为通知给adapter的ViewGroup的变动实现动画。它会自动的实现添加和删除的动画。这也不是一个简单的类，我们发现了`DefaultItemAnimator`用起来已经相当不错了。

##设置RecyclerView 
最后，如果你想初始化一个可用的`RecyclerView`，那么有以下的事情要做：

{% hightlight java %}
RecyclerView recyclerView = (RecyclerView) findViewById(R.id.list);
recyclerView.setHasFixedSize(true);
recyclerView.setAdapter(new MyRecyclerAdapter(createMockList(), R.layout.item));
recyclerView.setLayoutManager(new LinearLayoutManager(this));
recyclerView.setItemAnimator(new DefaultItemAnimator());
{% endhighlight %}

`setHasFixedSize()`方法是用来让RecyclerView的尺寸保持不变的。这些信息将被用于优化自身。

##总结
RecyclerView是个很有用并强大的View,学习路线可能会比较艰难，但是我相信Android组织会发布更好的LayoutManager的实现。

我创建了一个关于此示例的github仓库，我计划把它作为一个可扩展的类库的基础。你可以一个GridView的实现。

[RecyclerViewExtensions on Github](https://github.com/antoniolg/RecyclerViewExtensions)






