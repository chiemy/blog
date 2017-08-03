---
layout: post
title: "使用 SpannableString 简化布局"
modified: 2017-08-03 10:17:14
excerpt: "介绍 SpannableString 的使用方法，及使用 SpannableString 如何简化布局"
tags: [android, SpannableString]
published: true
---

<br>
## 什么是 SpannableString？

官方解释：This is the class for text whose content is immutable but to which markup objects can be attached and detached.

英语水平有限，只能翻译和理解前半句：文字内容不变，尝试翻译后半句：但是标记的对象可以关联和分离？

直接翻译很难理解了，只能透过现象来看了。最开始接触 ‘SpannableString’ 是为文字设置前景色（ForegroundColorSpan）或背景色（BackgroundColorSpan），我们知道文字的内容是不变的，但我们可以根据不同的 Span 让文字显示不同的效果。

## SpannableString 和 SpannableStringBuilder
`SpannableString` 和 `SpannableStringBuilder` 类似于 `String` 和 `StringBuilder`，`SpannableString` 内容不可变，`SpannableStringBuilder` 是可变的，可以通过 `append(CharSequence)` 方法添加文字。

## 使用

最重要的方法就是 `setSpan(Object what, int start, int end, int flags)` 方法了，该方法的作用是：为特定范围的文字标记特定的 Span。

参数说明：
* Object what: 用来标记的Span
* int start: 标记的起始位置
* int end: 标记的最终位置
* int flags: 

- `Spanned.SPAN_INCLUSIVE_INCLUSIVE`，闭区间[start, end] 
- `Spanned.SPAN_INCLUSIVE_EXCLUSIVE`，前闭后开[start, end)
- `Spanned.SPAN_EXCLUSIVE_INCLUSIVE`，前开后闭(start, end]
- `Spanned.SPAN_EXCLUSIVE_EXCLUSIVE`，开区间(start, end)

## Span

Android 系统提供了许多的 Span 样式能够满足我们的大部分需求，关于各种 Span 及用法可以看下这篇文章：[Android中各种Span的用法](http://blog.csdn.net/qq_16430735/article/details/50427978)总结的还是非常全面的。

如果这些都不能满足需求，我们还可以自定义 Span。关于自定义 Span 这块，还没做深入的研究，可以先参考以下文章：

[教你自定义android中span](http://blog.cgsdream.org/2016/07/06/custom-android-span/)
[Custom Colour Spans](https://blog.stylingandroid.com/custom-colour-spans/)

## 如何简化布局？
当然 SpannableString 的用处很多，这里举一个在开发中的小例子，使用 SpannableString 来简化布局。

在我的项目中会经常遇到如下布局：

![图1](http://ou3r6v4o4.bkt.clouddn.com/spannablestring01.png)

![图2](http://ou3r6v4o4.bkt.clouddn.com/spannablestring02.png)

数字、英文字母与汉字使用的是不同的字体，有时大小还可能不一致。

拿图1的布局举例，一般方法我们可能要写 6 个横向依次排列的 `TextView`，虽然能实现界面，但是显然是不能忍受的。还有一种方式就是自定义 View，但好像又没有必要，难道没有更简单的方式了吗？这时 `SpannableString` 就派上用场了。

使用 `SpannableString` 一个 `TextView` 就可以轻松搞定，但首先遇到的一个难点是，系统并没有提供标记字体的 Span，这时就需要自定义了，万能的 Google 搜一搜，轻松搞定（唉，不思进取，一定要写个自定义 Span 的文章），我把它放到了 gist 上：

<script src="https://gist.github.com/chiemy/3ad18a6e6d8ff44cfea22ec365a70d8c.js"></script>

另外附上，我在项目里用到的工具类，可设置文字颜色、大小、字体、样式，字体间加横向、纵向间距：

<script src="https://gist.github.com/chiemy/9030f3e89f0a4328b5f043b721475dd0.js"></script>

图1布局的实现：

```
TextView tv = ...;
tv.setTextColor(Color.WHITE);
Typeface typeface = 数字、英文用到的字体
CustomSpan span = new CustomSpan();
span.typeface("5", typeface)
    .append("分钟")
    .paddingRight(padding)
    .typeface("30", typeface)
    .append("千卡")
    .paddingRight(padding)
    .typeface("L1", typeface)
    .append("零基础")
    .applyTo(tv);
```

就是这样，一个 `TextView` 就可以轻松搞定类似的布局。


