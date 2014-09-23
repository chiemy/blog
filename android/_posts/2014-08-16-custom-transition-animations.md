---
layout: post
title: "[开发范例] 自定义过度动画"
modified: 2014-8-16 21:13:42
excerpt: "自定义Activity切换或者Fragment切换时的过度动画"
tags: [android, android开发范例]
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

##Activity过渡动画
有四种动画：打开一个新Activity时的进入和退出动画，当前Activity关闭时的进入动画和退出动画。

例如：当打开一个新的Activity时，当前Activity会进入“打开退出”动画，而新Activity将会进入“打开进入动画”,这些动画同时进行。

android系统自定义了许多动画，当然我们可以在res/anim文件夹下创建自定义动画。

可以在Activity的startActivity()或finish()之后，立即调用overridePendingTransition()方法，应用这些自定义动画。

也可以在定义的主题中，应用自定义动画：

{% highlight xml %}
<resources>
	<style name="AppTheme" parent="@android:Theme.Holo.Light">
		<item name="android:windowAnimationStyle">
			@style/ActivityAnimation
		</item>
	</style>

	<style name="ActivityAnimation" parent="@android:style/Animation.Activity">
		<item name="android:activityOpenEnterAnimation">
			@anim/动画资源
		</item>
		<item name="android:activityOpenExitAnimation">
			@anim/动画资源
		</item>
		<item name="android:activityCloseEnterAnimation">
			@anim/动画资源
		</item>
		<item name="android:activityCloseExitAnimation">
			@anim/动画资源
		</item>
	</style>
</resources>
{% endhighlight %}

>注意：样式一定要有Activity动画作为父类，因为以上四种动画并不是全部内容，如果不引入父类，窗口的一些其他动画效果就会消失了，除非你自定义。

##Fragment过渡动画
如果要实现Fragment过渡动画，那么有个前提，必须使用V4支持包。
使用支持库后，我们可以通过调用setCustomAnimations()方法，覆写单个FragmentTransaction的过渡动画。

此方法两个参数的版本，可以设置添加、剔除、移除时的动画效果，但在界面栈回退时不能设置相应的动画。

此四个参数的版本则可以为界面栈的回退添加自定义动画。

{% highlight java%}
fragTransaction.setCustomAnimations(……)
ft.replace(……);//或其他方法
ft.commit();
{% endhighlight %}

>注意调用顺序，要在add(),replace()和其他动作之前。

如果希望对某个Fragment一直使用同样的动画，需要覆写Fragment中的onCreateAnimation()方法,且FragmentTransaction要设置事务类型，那么就可以在onCreateAnimation方法中根据事务类型来显示相应的动画效果。

{% highlight java%}
//FragmentTransaction.TRANSIT_FRAGMENT_OPEN //开启
//FragmentTransaction.TRANSIT_FRAGMENT_CLOSE //关闭
//FragmentTransaction.TRANSIT_FRAGMENT_FADE //淡入
fragTransaction.setTransaction(FragmentTransaction.TRANSIT_FRAGMENT_OPEN);
ft.replace(……);
ft.commit();
{% endhighlight %}

如果是在Api 11或之后的版本中，Fragment是可以使用Animator对象的，Fragment的方法为onCreateAnimator()，而不是onCreateAnimation().

与Activity动画一样，也可以在主题中设置过渡动画，相应的属性分别为：
{% highlight xml %}
android:fragmentOpenEnterAnimation
android:fragmentOpenExitAnimation
android:fragmentCloseEnterAnimation
android:fragmentCloseExitAnimation
android:fragmentFadeEnterAnimation
android:fragmentFadeExitAnimation
{% endhighlight %}

>使用方法和Activity的类似，一样也不要忘记覆写父类主题。同时，在add,replace等操作之前，一定要设置事务，setTransaction