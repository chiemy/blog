---
layout: post
type: android
title: "[开发范例] 创建视图变换"
modified: 2014-8-17 22:48:07
excerpt: "视图转换"
tags: [android, android开发范例]
---

ViewGroup中的静态变换API提供了应用视觉效果的简单方法，例如旋转、缩放、透明度变化，而且不必依靠动画。使用它也很容易使用父视图的context来应用变换，比如根据位置的变化而缩放。

##实现方式

1. 调用`ViewGroup`的`setStaticTransformationsEnable(true)`;
2. 继承`ViewGroup`,覆写`getChildStaticTransformation(View view,Transformation t)`方法，根据条件对子视图进行处理;

如果要动态的改变子视图的效果，要禁止该视图的硬件加速功能（调用视图的`setLayerType(View.LAYER_TYPE_SOFTWARE,null)`方法，但实际测试中，有问题，只能在manifest中设置`android:hardwareAccelerated="false"`来禁用）

<img src="http://chiemyblog.qiniudn.com/device02.png" alt="Drawing" width="300" />

![Demos](http://chiemyblog.qiniudn.com/device01.gif)

源代码[https://github.com/chiemy/StaticTransformation](https://github.com/chiemy/StaticTransformation "源码")
