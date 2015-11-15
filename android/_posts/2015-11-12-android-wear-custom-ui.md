---
layout: post
title: "Android Wear应用布局的创建"
modified: 2015-11-11 14:51:03
excerpt: "本文将介绍如何创建Android wear布局及一些基本控件的使用"
tags: [android, android wear]
published: false
---
搭载Android Wear的手表现在已经很多了，屏幕有方的，有圆的。如何让界面在方形与圆形的设备上完美的展示，我想我们多少都有些困惑。还好，官方已经给出了比较完善的解决方案。

Android的一些基本控件我们是可以使用的，为了更好的适应Android Wear设备，还有些Android wear特有的控件，我们需要先了解一下。这些控件都在`com.google.android.support:wearable:+`支持包中。

### **WatchViewStub**

此控件用来为圆形、方形屏幕指定不同的布局。它是在运行时对屏幕的形状进行检测的，然后根据不同的屏幕形状填充不同的布局。

####用法：

首先我们将WatchViewStub作为Activity的根布局

	<android.support.wearable.view.WatchViewStub
    	xmlns:android="http://schemas.android.com/apk/res/android"
    	xmlns:app="http://schemas.android.com/apk/res-auto"
    	xmlns:tools="http://schemas.android.com/tools"
    	android:id="@+id/watch_view_stub"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent"
    	app:rectLayout="@layout/rect_activity_wear"
    	app:roundLayout="@layout/round_activity_wear">
	</android.support.wearable.view.WatchViewStub>
	
其中，`app:rectLayout`属性用来指定方形屏幕所引用的布局文件，`app:roundLayout`用于指定圆形屏幕所引用的布局文件。

由于是运行时填充的，那么布局里包含的控件我们需要在填充完毕后才能获取：

	@Override
	protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
    	setContentView(R.layout.activity_wear);

    	WatchViewStub stub = (WatchViewStub) findViewById(R.id.watch_view_stub);
    	stub.setOnLayoutInflatedListener(new 			WatchViewStub.OnLayoutInflatedListener() {
        		@Override 
        		public void onLayoutInflated(WatchViewStub stub) {
            		// 在这里我们可以获取相应的控件了
        		}
    	});
	}

###BoxInsetLayout
能够根据屏幕形状，进行自动调整的布局控件，使得在圆形屏幕下能够完整的显示布局。

我们通过为其子布局指定`app:layout_box`属性来指定显示区域，可能的取值及布局的位置：

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/uilib02.png" width="200"/>

- **<font color="#ff0000">top</font>** 图中，红线之下的区域

- **<font color="#00ff00">bottom</font>** 图中，绿线之上的区域

- **<font color="#0000ff">right</font>** 图中，蓝线右侧的区域

- **<font color="#FFD700">left</font>** 图中，黄线左侧的区域

- **all** 中间灰色区域

以上属性可组合使用。

使用示例：

{% highlight java %}
	<android.support.wearable.view.BoxInsetLayout
    	xmlns:android="http://schemas.android.com/apk/res/android"
    	xmlns:app="http://schemas.android.com/apk/res-auto"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent">

    	<FrameLayout
        	android:layout_width="match_parent"
        	android:layout_height="match_parent"
        	app:layout_box="all">

        	<TextView
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="#ff0000"
            android:text="@string/hello_round" />

    	</FrameLayout>
	</android.support.wearable.view.BoxInsetLayout>
{% endhighlight %}







	

