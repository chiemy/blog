---
layout: post
type: android
title: "[API Guide]App Resources-资源类型"
modified: 2014-9-16 21:55:28
excerpt: "常见不常用的资源类型用法总结"
tags: [android, api, resources, type]
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

##字符串资源

###String

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<string name="hello">Hello!</string>
</resources>
{% endhighlight %}

>注：Context中的getString()和getText()都可以获得字符串，但getText()方法可以获得格式化的字符串。
比如将上边的改为<string name="hello"><b>Hello!</b></string>
那么getString()方法，获得的只是Hello!，而getText()获取的是粗体的Hello!

###String Array

略

###Quantity Strings (Plurals)

不同的语言对数量进行描述的语法规则也不同。比如在英语里，数量1是个特殊情况，我们写成“1 book”，但其他任何数量都要写成“n books”。这种单复数之间的区别是很普遍的，不过其他语言会有更好的区分方式。Android支持的全集包括zero、one、 two、few、many和other。决定选择和使用某种语言和复数的规则是非常复杂的，所以Android提供了诸如getQuantityString()的方法来选择合适的资源。

如果要保持应用程序的风格一致，常可以用诸如“Books: 1”的模糊数量形式来避免使用数量字符串。

语法

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<plurals
 		name="plural_name">
 		<item
 		quantity=["zero" | "one" | "two" | "few" | "many" | "other"]
 			>text_string</item>
 	</plurals>
</resources>
{% endhighlight %}
 
quantity的取值

<div class="row">
    <div class="large-12 columns">
 <table style="border-spacing: 0px;margin: 4px 4px; width: 100%; border-left:1px solid #ccc;border-top:1px solid #ccc;"> 
    <tr style="background:#DEE8F1;"> 
     <th style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px"> Class/Interface </th> 
     <th style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px"> Description </th>
    </tr> 
    <tr style="vertical-align:top;"> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>zero </p> </td> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>语言需要对数字0进行特殊处理。（比如阿拉伯语） </p> </td>
    </tr> 
    <tr style="vertical-align:top;"> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>one </p> </td> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>语言需要对类似1的数字进行特殊处理。（比如英语和其它大多数语言里的1；在俄语里，任何以1结尾但不以11结尾的数也属于 </p><p>此类型。） </p> </td>
    </tr> 
    <tr style="vertical-align:top;"> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>two </p> </td> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>语言需要对类似2的数字进行特殊处理。（比如威尔士语） </p> </td>
    </tr> 
    <tr style="vertical-align:top;"> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>few </p> </td> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>语言需要对较小数字进行特殊处理（比如捷克语里的2、3、4；或者波兰语里以2、3、4结尾但不是12、13、14的数。） </p> </td>
    </tr> 
    <tr style="vertical-align:top;"> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>many </p> </td> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>语言需要对较大数字进行特殊处理（比如马耳他语里以11-99结尾的数） </p> </td>
    </tr> 
    <tr style="vertical-align:top;"> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>other </p> </td> 
     <td style="border-right:1px solid #ccc;border-bottom:1px solid #ccc; padding:5px 15px;"> <p>语言不需要对数字进行特殊处理。 </p> </td>
    </tr>
  </table>
    </div>
</div>

举例

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<plurals name="numberOfSongsAvailable">
 		<item quantity="one">One song found.</item>
 		<item quantity="other">%d songs found.</item>
 	</plurals>
</resources>
{% endhighlight %}

代码中

{% highlight java %}
int count = getNumberOfsongsAvailable();
Resources res = getResources();
String songsFound = res.getQuantityString(R.plurals.numberOfSongsAvailable, count, count);
{% endhighlight %}

首先，根据第二个参数值，匹配到相应的quantity，第三个参数会替代对应的字符串中%d的位置，如果没有%d等，第三个参数可省略

###格式化问题

如果字符串中有“省略号”，“引号”则需要进行转意
举例：：

{% highlight xml %}
<string name="good_example">"This'll work"</string> //起作用
<string name="good_example_2">This\'ll also work</string>//起作用
<string name="bad_example">This doesn't work</string> //不起作用
<string name="bad_example_2">XML encodings don&apos;t work</string> //不起作用
{% endhighlight %}

举例：

`<string name="welcome_messages">你好, %1$s! 你有 %2$d 条新消息.</string>`

 %1$s 代表可被字符串替换，%2$d 表示可被数字替换

代码中：

{% highlight java %}
Resources res = getResources();
String text = String.format(res.getString(R.string.welcome_messages), “chiemy”, 10);
{% endhighlight %}

那么text的内容为：你好，chiemy！你有10条新消息！

####使用Html标记样式

举例：

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<string name="welcome">Welcome to <b>Android</b>!</string>
</resources>
{% endhighlight %}

支持的Html元素有：

- `<b>`粗体
- `<i>`斜体
- `<u>`下划线

如果要使用`String.format`格式带有`Html`标记的字符串，则需要对标记进行转意

例如

{% highlight xml %}
<resources>
  <string name="welcome_messages">Hello, %1$s! You have &lt;b>%2$d new messages&lt;/b>.</string>
</resources>
{% endhighlight %}

格式化前，必须保证将 像"<" or "&"这样的符号进行转意，才能保证`Html.fromHtml`方法得到的是Html格式的字符串

{% highlight java %}
Resources res = getResources();
String text = String.format(res.getString(R.string.welcome_messages), username, mailCount);
CharSequence styledText = Html.fromHtml(text);
{% endhighlight %}

因为`fromHtml(String)`方法会格式化所有的HTML内容，所以要确保用`htmlEncode(String)`对带格式化文本的字符串内所有可能的HTML字符进行转义。

比如，如果要把可能包含诸如“<”或“&”等字符的串作为参数传给`String.format()`，那么在格式化之前必须对这些字符进行转义。格式化之后再把字符串传入`fromHtml(String)`，这些特殊字符就能还原成本来意义了。例如：

{% highlight java %}
String escapedUsername = TextUtil.htmlEncode(username);

Resources res = getResources();
String text = String.format(res.getString(R.string.welcome_messages), escapedUsername, mailCount);
CharSequence styledText = Html.fromHtml(text);
{% endhighlight %}

##TypeArray-类型数组

语法

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<array
 	name="integer_array_name">
 		<item>resource</item>
 	</array>
</resources>
{% endhighlight %}

举例

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<array name="icons">
 		<item>@drawable/home</item>
 		<item>@drawable/settings</item>
 		<item>@drawable/logout</item>
 	</array>
 	<array name="colors">
 		<item>#FFFF0000</item>
 		<item>#FF00FF00</item>
 		<item>#FF0000FF</item>
 	</array>
</resources>
{% endhighlight %}

java代码

{% highlight xml %}
Resources res = getResources();
TypedArray icons = res.obtainTypedArray(R.array.icons);
Drawable drawable = icons.getDrawable(0);

TypedArray colors = res.obtainTypedArray(R.array.colors);
int color = colors.getColor(0,0);
{% endhighlight %}