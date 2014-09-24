---
layout: post
title: "[API Guide]User Interface-样式和主题"
modified: 2014-9-16 21:20:58
excerpt: "样式和主题的使用"
tags: [android, Android Api, 翻译, User Interface]
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

[官方Api Guide链接](http://developer.android.com/guide/topics/ui/themes.html)

样式(Style)是用来指定View或者window的外观和格式的一组属性集合。可以用来指定高度、内边距、字体颜色、字体大小、背景颜色等属性。样式定义在独立于布局文件的XML文件中。保证了内容和设计的独立性。

例如

{% highlight xml %}
<TextView
 	android:layout_width="fill_parent"
 	android:layout_height="wrap_content"
 	android:textColor="#00FF00"
 	android:typeface="monospace"
 	android:text="@string/hello" />
{% endhighlight %}

应用style文件后

{% highlight xml %}
<TextView
 	style="@style/CodeFont"
 	android:text="@string/hello" />
{% endhighlight %}

所有和样式相关的属性都被移出了布局文件，放到了叫做CodeFont的样式文件中。

主题（theme）是与Activity或者应用相关的样式，当样式作为主题存在时，Activity或者应用中的所有View的相关属性会随其指定的样式改变。例如，当CodeFont样式文件作为一个Activity的主题的时候，那么其中所有的字体都会变成CodeFont所声明的那样。

##样式的定义
样式文件保存在`res/values/`目录下，文件名可以是任意的，但是必须是`.xml`文件。
根节点必须是`<resources>`标签

一个样式对应一个`<style>`标签，并且为其指定唯一的name值。添加一个`<item>`元素对应样式的一个属性，name为样式属性的名字，后边对应其属性的值。`<item>`的值可以是字符串，16进制颜色，对其他资源的引用，或者其他样式属性的值。下面是例子：

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<style name="CodeFont" parent="@android:style/TextAppearance.Medium">
 	<item name="android:layout_width">fill_parent</item>
 		<item name="android:layout_height">wrap_content</item>
 		<item name="android:textColor">#00FF00</item>
 		<item name="android:typeface">monospace</item>
 	</style>
</resources>
{% endhighlight %}

每个`<resources>`的子元素都会在编译器被转换为应用资源对象，可以被`<style>`的name属性引用。

`<style>`标签中的parent属性是可选的，可以从其指定的资源ID中继承一些属性，这个属性也可以被重写。

作为`Activity`或应用的主题的样式文件，和上边的用法是相同的。

###继承
`<style>`标签中的`parent`属性，可以指定一个`style`,并从中继承属性。你可以用他来从已有的样式中继承一些属性，然后定义一些你想改变或添加的属性。你可以继承自定义的样式文件，也可以继承创建在平台（platform）中的样式文件。

例如，你可以继承安卓平台默认的文字属性：

{% highlight xml %}
<style name="GreenText" parent="@android:style/TextAppearance">
 	<item name="android:textColor">#00FF00</item>
</style>
{% endhighlight %}

如果继承自定义的样式文件，就不需要parent属性，只需要在新的样式名称前边加上要继承的样式名称作为前缀，以“.”分隔。

例如继承上边的CodeFont,并把字体换成红色：

{% highlight xml %}
<style name="CodeFont.Red">
 	<item name="android:textColor">#FF0000</item>
</style>
{% endhighlight %}

也可以继续向下继承，如CodeFont.Red.Big……

>注意：此方法只适用于自定义样式，不能应用与Android内置的样式文件，而必须用parent属性来实现继承。

###样式属性
在了解了如何定义样式之后，接下来学习哪类样式属性是可用的。对其中的一些我们已经很熟悉了，比如`layout_width,textColor`。当然，还有很多我们能够使用的属性。
我们可以在与View相应的XML标签下找到属性信息，例如，`TextView` 的XML下的属性可以用于为`TextView`元素定义的样式中（或其子类）。其中有`android：inputType`这个属性，因此我们可以在样式中定义`name`为`android：inputType`的元素,如下：

`EditText`有标签下有`inputType`属性

{% highlight xml %}
<EditText
 	android:inputType="number"
 	... />
{% endhighlight %}

那么可以在样式文件中声明为：

{% highlight xml %}
<style name="Numbers">
  <item name="android:inputType">number</item>
  ...
</style>
{% endhighlight %}

我们就可以将EditText替换成这个样子：

{% highlight xml %}
<EditText
 	style="@style/Numbers"
 	... />
{% endhighlight %}

记住`View`对象接收的样式属性不完全相同，如果样式中的一些属性不被`View`对象所支持，那么不支持的属性就会被`View`忽略掉。

一些属性信息，只是用于主题，这些属性与整个窗口（window）关联，不与任何View关联。
例如，一些样式属性，可以隐藏应用的标题，隐藏状态栏，或者改变窗口的背景。这些属性就不属于任何`View`对象。

例如，`windowNoTitle`和`windowBackground`属性仅当样式应用于`Activity`或`Application`的时候才会起效。

##为UI设置样式和主题

###为视图设置样式
略

###为Activity或application设定主题
为应用中所有Activity设定主题，可以在`AndroidManifest.xml`文件下的`<application>`标签添加`android:theme`属性：
如：`<application android:theme="@style/CustomTheme">`

如果只为某个Activity添加主题，只需要在相应的Activity标签下，添加`android:theme`属性即可。

如果你喜欢某个主题，但是想对其做一些调整。那么就添加此主题作为你的自定义主题的parent，例如：你可以更改`android:Theme.light`主题为自己想要的颜色，就像这样

{% highlight xml %}
<color name="custom_theme_color">#b0b0ff</color>
<style name="CustomTheme" parent="android:Theme.Light">
 	<item name="android:windowBackground">@color/custom_theme_color</item>
 	<item name="android:colorBackground">@color/custom_theme_color</item>
</style>
{% endhighlight %}

>注意：`windowBackground`的颜色必须是对其他资源的应用，不能直接指定颜色，而`colorBackgroud`则可以直接指定

###基于平台选择主题
新版本的Android应用程序提供额外的theme，你可能想使用它们在这些平台上运行，同时与旧版本兼容。你可以通过使用自定义theme资源选择不同的parent theme，根据平台版本之间切换完成。

例如，这里声明一个自定义theme，相当于是标准平台上默认的light theme。它将在XML文件的res/values 目录下（通常是res/values/styles.xml ）：

{% highlight xml %}
<style name="LightThemeSelector" parent="android:Theme.Light">
	...
</style>
{% endhighlight %}

当程序运行在Android3.0（API等级11）或更高的版本时使用新的holographic theme，你可以在res/values-v11 的XML文件中放置另一个声明theme，但holographic theme的parent theme像这样设置：

{% highlight xml %}
<style name="LightThemeSelector" parent="android:Theme.Holo.Light">
	...
</style>
{% endhighlight %}

现在可以如其他的theme那样使用这个theme，如果你的应用程序运行在Android3.0或更高的版本时，将自动切换到holographic theme。
你可以在R.styleable.Theme中找到你能够使用的theme的标准属性列表。

获得更多关于如theme和layout提供替代资源，基础平台版本或其他设备配置的详细信息，请参阅Providing Resources文档。

##使用平台的样式和主题

Android平台提供了大量的style和theme供你在应用程序中使用。你可以在R.style类中找到所有可用的style。要使用这些style，用句点替换style名称中的下划线。例如，你可以通过"@android:style/Theme.NoTitleBar"应用Theme_NoTitleBar theme。

然而，R.style没有好的文档，没有详细叙述这些style，所以查看这些style和theme的实际源代码将使你更好理解每个style属性提供了什么功能。为更好参考Android的style和theme，请参阅下列源代码：

- [Android Styles (styles.xml)](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/res/res/values/styles.xml)
- [Android Themes (themes.xml)](https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/res/res/values/themes.xml)

这些文件将通过例子帮助你学习。举个例子，在Android theme源代码中，你将会找到一个`<style name="Theme.Dialog">`声明。在这个定义中，你将看到所有由Android框架使用的用于对话框的style属性。

为获得更多关于在XML中创建style的语法，参阅[Style Resource](http://developer.android.com/guide/topics/resources/style-resource.html)。

关于你可以用来定义style或theme的可用style属性（例如，"windowBackground" 或 "textAppearance"），参阅[R.attr](http://developer.android.com/reference/android/R.attr.html)或者对应于你正在为其创建一个style的View类。