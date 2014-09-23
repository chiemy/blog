---
layout: post
type: android
title: "[API Guide]App Resources-处理运行时的变化"
modified: 2014-9-16 21:00:36
excerpt: "在运行时，对配置改变的处理"
tags: [android, api, resources, runtime change]
comments: true
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

有时保存引用程序状态代价是很高的，那么有两种方法解决这个问题：

1. 在配置改变时，保留一个对象
2. 自己控制参数的改变

##保留对象
如果重新启动一个Activity，需要你恢复大量的数据， 从新建立网络连接，或者其他密集的操作，那么配置的改变，可能会带来不好的用户体验。

同时，使用Bundle对象保存全部的Activity的状态也是不可能的事情，Bundle不是为保存大对象（比如位图）而设计的，而且保存的对象也必须是可序列化的，这样势必会消耗大量的内存，使得配置的变化很慢。在这种情况下，你可以通过保留一个Fragment，来减少重新初始化activity带来的负担。这个Fragment会保持对状态性对象的引用。

在配置更改时在Fragment中保存对象

1. 扩展片段类，并且声明引用状态对象。
2. 创建片段时调用setRetainInstance(boolean)。
3. 向Activity中添加片段。
4. 活动重启时使用FragmentManager来检索片段。

<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=RetainedFragment.java"></script>

然后使用FragmentManager向活动添加片段。运行配置更改时活动重启，就可以从片段获取数据对象。例如按照如下方式定义活动：

<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=MyActivity.java"></script>

此例中onCreate()添加片段或者恢复引用。onCreate()方法中添加了Fragment(存储状态对象)。onDestroy()更新了保留的Fragment中的状态对象。

旧版Api中不是通过Fragment,而是以下方法

<s>1. 重写onRetainNonConfigurationInstance() 方法返回你要保存的对象</s>

<s>2. 当activity重新创建时，调用getLastNonConfigurationInstance() 方法恢复保存的对象。</s>


当由于配置改变使得系统将你的Activity关闭的时候，会在onStop()和onDestroy()之间调用onRetainNonConfigurationInstance() 方法，你可以返回一个对状态恢复比较重要的对象。

>注意：不能返回跟Context相关的内容， 比如Drawable,Adapter, View 。如果这样做的话，会导致Leak resource,意味着你的应用保持一个指向他们的引用，使得垃圾回收器不能回收他们，会损失许多内存资源。

##自己来处理配置的变化
如果你不想在特定的配置改变时，让你的应用更新资源，避免activity的重建，你可以声明activity处理这些配置更改的事件，这样可以阻止Activity重新创建。

>特别提醒: 选择自己来处理配置的变化会使得可替代资源的使用变得更困难，因为系统不会为你来自动调用这些资源。这种技术应该被视为最后的手段，对于大多数应用程序不建议使用。

方法：在`<activity>`标签下 加入`android:configChanges` 属性，属性值代表你要控制的配置，

包括如下值：

- "mcc"
- "mnc"
- "locale"
- "touchscreen"
- "keyboard"
- "keyboardHidden"
- "navigation"
- "screenLayout"
- "fontScale"
- "uiMode"
- "orientation"
- "screenSize"
- "smallestScreenSize"
- "layoutDirection"

需要注意的是，从Android 3.2开始，当屏幕方向改变时，screen size也会改变，因此，如果你想要控制屏幕方向的配置，那么就必须加上screenSize值，android:configChanges="orientation|screenSize".

下面的方法是检测当前屏幕方向的:

{% highlight java %}
@Override
public void onConfigurationChanged(Configuration newConfig) {
 	super.onConfigurationChanged(newConfig);
 	// Checks the orientation of the screen
 	if (newConfig.orientation == Configuration.ORIENTATION_LANDSCAPE) {
 		Toast.makeText(this, "landscape", Toast.LENGTH_SHORT).show();
 	} else if (newConfig.orientation == Configuration.ORIENTATION_PORTRAIT){
 		Toast.makeText(this, "portrait", Toast.LENGTH_SHORT).show();
 	}
}
{% endhighlight %}

Configuration代表所有的配置对象，并不只代表改变的配置。

>记住：如果你的Activity处理了某项配置更改的事件，那么你就要为元素重新分配替代资源。例如，Activity处理屏幕方向改变的配置，那么你就应该在onConfigurationChanged()方法中为元素重新分配资源。

如果配置改变时，只是为了不让Activity重新创建，而不使用替代资源，那么就不用重写onConfigurationChanged()方法。

但是，你的应用总是能够在某些条件下能够关闭和重启，所以不应该在Activity生命周期中，把这项技术作为逃避保存应用状态的借口。不仅是因为，总会有其他的一些配置使得你不能阻止应用的重启，还因为，你应该在如用户离开应用，回到应用之前处理一些事件。