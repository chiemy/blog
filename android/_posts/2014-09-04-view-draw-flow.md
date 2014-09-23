---
layout: post
type: android
title: "View绘制流程"
modified: 2014-9-9 14:39:24
excerpt: "绘制流程及各主要函数的简单介绍"
tags: [android, view, measure, layout, draw]
comments: true
---


整个View树的绘图流程是在`ViewRoot`类的`performTraversals()`函数展开的，其流程大致如下：

<figure>
	<a href="http://chiemyblog.qiniudn.com/view%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B.png"><img src="http://chiemyblog.qiniudn.com/view%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B.png"></a>
	<figcaption><a href="http://chiemyblog.qiniudn.com/view%E7%BB%98%E5%88%B6%E6%B5%81%E7%A8%8B.png" title="view绘制流程">View绘制流程图</a></figcaption>
</figure>

说明：

1. 一个`activity`包含有一个`PhoneWiondow`对象，而所有的UI部件都是放在`PhoneWiondow`中。ViewRoot这个类在android的UI结构中扮演的是一个中间者的角色，连接的是`PhoneWindow`跟`WindowManagerService`.
2. 每一个`PhoneWiondow`中都有一个叫`DecorView`的对象(`PhoneWiondow`的内不类，并且是其子类)，同时也是`FrameLayout`的子类，是所有窗口(一个Dialog，一个Toast，一个Menu菜单)的根View
3. `DecorView`结构，有两个FrameLayout，一个是标题栏，一个是要填充的视图

![DecorView结构](http://chiemyblog.qiniudn.com/DecorView.png)


![关联](http://chiemyblog.qiniudn.com/%E5%85%B3%E8%81%94%E5%9B%BE.png)
	



##measure()过程

![measure流程](http://chiemyblog.qiniudn.com/measure%E6%B5%81%E7%A8%8B.png)

###1.作用

为整个View树计算实际的大小，即设置实际的高(对应属性:mMeasuredHeight)和宽(对应属性:mMeasureWidth)，每个View的控件的实际宽高都是由父视图和本身视图决定的

###2.操作

在此方法中，调用子视图的`onMeasure()`方法，`onMeasure()`方法中的操作包括：

- 通过调用该视图的`setMeasuredDimension()`方法设置实际的宽高。

- 如果该视图是`ViewGroup`类型，要对子视图遍历进行`measure`操作，对子视图的`measure`操作是通过调用`measureChildWithMargins()`方法实现的，该方法内部只是简单的调用了View对象的`measure()`方法 

通过伪代码的方式描述如下：

{% highlight java%}
//回调View视图里的onMeasure过程
private void onMeasure(int height , int width){
   //设置该view的实际宽(mMeasuredWidth)高(mMeasuredHeight)
   //1、该方法必须在onMeasure调用，否者报异常。
   setMeasuredDimension(h , l);
	
   //2、如果该View是ViewGroup类型，则对它的每个子View进行measure()过程
   int childCount = getChildCount() ;   
   for(int i=0 ;i<childCount ;i++){
   		//2.1、获得每个子View对象引用
   		View child = getChildAt(i) ;
   		//整个measure()过程就是个递归过程
   		//该方法只是一个过滤器，最后会调用measure()过程 ;或者 measureChild(child , h, i)方法都
   		measureChildWithMargins(child , h, i) ; 
			   
  		//其实，对于我们自己写的应用来说，最好的办法是去掉框架里的该方法，直接调用view.measure()，如下：
 		//child.measure(h, l)
  	}
}
	
//该方法具体实现在ViewGroup.java里 。
protected  void measureChildWithMargins(View v, int height , int width){
 	v.measure(h,l)   
}
{% endhighlight %}

##layout布局过程

![layout流程](http://chiemyblog.qiniudn.com/layout%E6%B5%81%E7%A8%8B.png)

###1.作用

为将整个根据子视图的大小以及布局参数将View树放到合适的位置上

###2.操作过程

view中的该layout函数流程大概如下：

1. 调用`setFrame()`将位置参数保存，这些参数会保存到view内部变量（mLeft,mTop,mRight,mButtom）。如果有数值的变化那么会调用`invalidate()`重绘那些区域。
2. 回调`onLayout()`，`View`中定义的`onLayout（）`函数默认什么都不做，View系统提供onLayout函数的目的是为了使系统包含有子视图的父视图能够在`onLayout()`函数对子视图进行位置分配，正因为这样，如果是`viewGroup`类型，就必须重载`onLayout()`，由此`ViewGroup`的`onLayout`为`abstract`也就很好解释了

{% highlight java %}
// layout()过程  ViewRoot.java
// 发起layout()的"发号者"在ViewRoot.java里的performTraversals()方法， mView.layout()
private void  performTraversals(){
   //...
   View mView;
   mView.layout(left,top,right,bottom);
   	
   //....
}
 
//回调View视图里的onLayout过程 ,该方法只由ViewGroup类型实现
private void onLayout(int left , int top , right , bottom){
	 
   //如果该View不是ViewGroup类型
   //调用setFrame()方法设置该控件的在父视图上的坐标轴   
   setFrame(l ,t , r ,b) ;   
   //--------------------------   
   //如果该View是ViewGroup类型，则对它的每个子View进行layout()过程
   int childCount = getChildCount() ;   
   for(int i=0 ;i<childCount ;i++){
 		//2.1、获得每个子View对象引用
 		View child = getChildAt(i) ;
 		//整个layout()过程就是个递归过程
 		child.layout(l, t, r, b) ;
 	}
}
{% endhighlight %}

##draw过程

这是见证奇迹的时刻，`draw`过程就是要把view对象绘制到屏幕上，如果它是一个`viewGroup`，则需要递归绘制它所有的子视图。视图中有哪些元素是需要绘制的呢？

1. view背景。所有view都会有一个背景，可以是一个颜色值，也可以是一张背景图片
2. 视图本身内容。比如`TextView`的文字
3. 渐变边框。就是一个shader对象，让视图看起来更具有层次感
4. 滚动条

![layout流程](http://chiemyblog.qiniudn.com/draw%E6%B5%81%E7%A8%8B.png)

上面这张图比前面的measure和layout多了一步：draw（）。在performTraversals（）函数中调用的是viewRoot的draw（）函数，在该函数中进行一系列的前端处理后，再调用host.draw()。

一般情况下，View对象不应该重载draw（）函数，因此，host.draw（）就是view.draw()。该函数内部过程也就是View系统绘制过程的核心过程，该函数中会依次绘制前面所说哦四种元素，其中绘制视图本身的具体实现就是回调onDraw()函数，应用程序一般也会重载onDraw()函数以绘制所设计的View的真正界面内容。

{% highlight java %}
// draw()过程     ViewRoot.java
// 发起draw()的"发号者"在ViewRoot.java里的performTraversals()方法， 该方法会继续调用draw()方法开始绘图
private void  draw(){
 	//...
 	View mView;
 	mView.draw(canvas);  
 	//....
}
   
//回调View视图里的onLayout过程 ,该方法只由ViewGroup类型实现
private void draw(Canvas canvas){
 	//该方法会做如下事情
 	//1 、绘制该View的背景
 	//2、为绘制渐变框做一些准备操作
 	//3、调用onDraw()方法绘制视图本身
 	//4、调用dispatchDraw()方法绘制每个子视图，dispatchDraw()已经在Android框架中实现了，在ViewGroup方法中。
 	// 应用程序程序一般不需要重写该方法，但可以捕获该方法的发生，做一些特别的事情。
 	//5、绘制渐变框	
}
   
//ViewGroup.java中的dispatchDraw()方法，应用程序一般不需要重写该方法
@Override
protected void dispatchDraw(Canvas canvas) {
   // 
   //其实现方法类似如下：
   int childCount = getChildCount() ;   
   for(int i=0 ;i<childCount ;i++){
 		View child = getChildAt(i) ;
 		//调用drawChild完成
 		drawChild(child,canvas) ;
 	}	   
}
//ViewGroup.java中的dispatchDraw()方法，应用程序一般不需要重写该方法
protected void drawChild(View child,Canvas canvas) {
   // ....
   //简单的回调View对象的draw()方法，递归就这么产生了。
   child.draw(canvas) ; 
   //.........
}
{% endhighlight %}

绘制完界面内容后，如果该视图还包含子视图，则调用`dispatchDraw()`函数，实际上起作用的还是`viewGroup`的`dispatchDraw()`函数。需要说明的是应用程序不应该再重载`ViewGroup`中该方法，因为它已经有了默认而且标准的view系统流程。`dispatchDraw()`内部for循环调用`drawChild()`分别绘制每一个子视图，而`drawChild()`内部又会调用`draw（）`函数完成子视图的内部绘制工作。


参考：
	
[戏说Android view 工作流程《上》](http://blog.csdn.net/aaa2832/article/details/7844904)

[戏说Android view 工作流程《下》](http://blog.csdn.net/aaa2832/article/details/7849400)
