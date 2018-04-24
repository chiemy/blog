---
layout: post
title: "用PageTranformer实现更炫酷的动画"
modified: 2014-9-11 15:38:18
excerpt: "ViewPager.PageTranformer的基本用法及扩展"
tags: [android, ViewPager, Animation]
comments: true
---

## PageTransformer的基本使用
-----
`ViewPager`在开发中应用的非常普遍，一些基本用法都已经掌握了，但是有个叫`PageTransformer`的东东，还是第一次接触，用起来的确非常酷。

`PageTransformer`是一个允许你修改`ViewPager`默认滑动动画的接口，通过它可以实现一些非常炫酷的滑动动画。

那么它是如何工作的呢？[官方文档](http://developer.android.com/training/animation/screen-slide.html#pagetransformer)已经有比较详细的说明了，所以说看文档还是很有必要的。其实使用起来也非常简单：

- 实现`ViewPager.PageTransformer`接口
- 实现`transformPage(View view,float position)`

其中`transformPage()`方法中的参数`View`很好理解，就是显示在屏幕中的`page`(所以这个view是随着滑动状态的变化而变化的哦)，
至于这个`position`参数是什么意思呢？通过一张图来说明:

<figure>
	<img src="http://chiemyblog.qiniudn.com/page_transformer1.png">
	<figcaption>position参数</figcaption>
</figure>

屏幕(确切的说，应该是ViewPager)左边缘为0，右边缘为1，page左边缘的所处的位置，就是此page的`position`的取值了。

以上图为例,page-0,1,2的当前`position`值分别为-1、0、1，如果从当前位置滑向page-0，并最终到page-0，那么page-0的`position`值会从-1逐渐增大到0，page-1的`position`值会从0逐渐增大到1，page-2的position值会从1逐渐增大。滑动过程中会回调`transformPage()`方法，我们就可以根据`position`的值实现一些动画效果了。

## 不止如此，还有……
-----
`transformPage()`方法提供了`position`的值和page所对应的`view`，一般情况下，我们会对此`view`进行动画处理，但是不限于此。
我们可以对此`view`的内部的`view`进行动画处理，就像下边这样:

<figure>
	<img src="https://d262ilb51hltx0.cloudfront.net/max/600/1*zD4p2a5gBqt63PQH9ZLNdQ.gif">
	<figcaption>动态效果图</figcaption>
</figure>


代码示例：

<script src="https://gist.github.com/anonymous/d93f16ea8f5639615cce.js"></script>

工程示例[PageTransformerDemo](https://github.com/chiemy/PageTransformerDemo)

参考

[Great animations with PageTransformer](https://medium.com/@BashaChris/the-android-viewpager-has-become-a-fairly-popular-component-among-android-apps-its-simple-6bca403b16d4 "Great animations with PageTransformer")

