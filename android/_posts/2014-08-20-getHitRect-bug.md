---
layout: post
type: android
title: "View的getHitRect()方法的bug"
modified: 2014-8-20 10:41:22
excerpt: "Strange View.getHitRect() behaviour"
tags: [android, problem]
comments: true
---

在Android 4.3以下的版本中，View的getHitRect()方法有bug，在某些情况下获取的Rect并不准确，可以用以下方法代替：

{% highlight java %}
public Rect getHitRect(View child){
   Rect frame = new Rect();
   frame.left = child.getLeft();
   frame.right = child.getRight();
   frame.top = child.getTop();
   frame.bottom = child.getBottom();
   return frame;
}
{% endhighlight %}

[Strange View.getHitRect() behaviour](http://stackoverflow.com/questions/17750116/strange-view-gethitrect-behaviour "StackOverflow")