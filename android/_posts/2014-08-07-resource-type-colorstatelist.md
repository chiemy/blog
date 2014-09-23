---
layout: post
type: android
title: "[Resource Type] ColorStateList"
modified: 2014-08-08
excerpt: "ColorlStateList的使用"
tags: [android, api, resource, 翻译]
---

ColorStateList是可以作为颜色定义在xml中的一个对象，它会根据与之关联的View对象的状态，改变颜色。在xml文件中，定义在一个<selector>标签下的，每个<item>标签代表一种颜色，通过多个属性值对它的状态，以及触发的条件进行描述。

`<item>`标签属性

{% highlight xml %}
android:color
android:state_pressed
android:state_focused
android:state_selected
android:state_checkable
android:state_checked
android:state_enabled
android:state_window_focused
当为true时，表示应用在前台时使用此值，
当为false时，表示应用的窗口失去焦点时（例如下拉通知栏、出现dialog窗口）使用自属性值。
{% endhighlight %}
   
> 注意：在状态列表中第一个匹配当前状态的Item会被使用，因此如果列表中第一项没有包含上边任何属性的话,那么会每次都被使用。这就是默认的值必须放在列表最后的原因。

