---
layout: post
title: "ViewPager使用中的一些问题"
modified: 2014-8-19 21:13:42
excerpt: ""
tags: [android, ViewPager]
---
<section id="table-of-contents" class="toc">
  <header>
    <h3>主要内容</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

##ViewPager的刷新问题

在ViewPager使用过程中，会有对某个界面或全部界面进行刷新的需求，和`BaseAdapter`等类似，`PagerAdapter`也有`notifyDataSetChanged()`方法，但是发现调用此方法并不起作用。

其实`notifyDataSetChanged()`只会在ViewPager的基础数据发生变化时才会起作用，例如，当你动态的添加/删除页面（从list中添加/删除item）时。

此时，ViewPager会根据`getItemPosition()`方法和`getCount()`方法，来决定是否创建或删除视图，如果对于相应位置的子视图`getItemPosition()`返回的是`POSITION_NONE`,ViewPager会认为该子视图需要被删除，然后调用`destroyItem()`方法，删除视图。

因此，如果你只是想更新特定的视图，那么覆写`getItemPosition()`返回`POSITION_NONE`，调用`notifyDataSetChanged()`可以达到更新的目的，但是那些不需要更新的视图也会被重新加载，如果子视图是简单的视图（如TextView）那么看似没什么问题，但如果是复杂的视图（例如从数据库构成的ListView），那么会造成很大的资源消耗。

那么有没有更好的方法呢？

其中一个较好的方法是：在`instantiateItem()`方法中，为每个初始化的View设置Tag，即使用`setTag()`方法，当我们要改变某个视图的数据时，通过调用`ViewPager`的`findViewWithTag()`方法，找到相应视图，这样我们就可以重新利用该视图达到刷新目的，而避免不必要的资源消耗。

参考：[StackOverflow](http://stackoverflow.com/questions/7263291/viewpager-pageradapter-not-updating-the-view/8024557#8024557 "参考")

##对isViewFromObject()方法理解

该方法声明了返回值不一定是view，可以是任意对象。要知道view的添加是在`instantiateItem`方法内部，通过container来添加的，所以这个方法不一定要返回view。

而`isViewFromObject`方法是用来判断pager的一个view是否和`instantiateItem`方法返回的object有关联，如果有关联做什么呢？去看ViewPager源码，你去看下`addNewItem`方法，会找到 `instantiateItem` 的使用方法，注意这里的`mItems`变量。然后你再搜索`isViewFromObject`，会发现其被`infoForChild`方法调用，返回值是`ItemInfo`。再去看下`ItemInfo`的结构，其中有一个`object`对象，该值就是`instantiateItem`返回的。

也就是说，`ViewPager`里面用了一个`mItems(ArrayList)`来存储每个page的信息(ItemInfo)，当界面要展示或者发生变化时，需要依据page的当前信息来调整，但此时只能通过view来查找，所以只能遍历mItems通过比较view和object来找到对应的`ItemInfo`。

说的有些乱，好好看源码就懂了！


