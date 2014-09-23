---
layout: post
type: android
title: "[API Guide]App Resources-资源的访问"
modified: 2014-9-16 21:40:21
excerpt: "访问资源的方法"
tags: [android, api, resources, access]
comments: true
---

两种访问方式：

代码中：R类的静态整形变量
	
例如：R.string.hello，string代表资源类型，hello代表资源名

xml中：使用特殊的xml语法，例如@string/hello


##通过代码访问

语法

`[<package_name>.]R.<resource_type>.<resource_name>`

`<package_name>`当引用的是自己应用中的资源时，可以省略

##在xml中访问
语法

`@[<package_name>:]<resource_type>/<resource_name>`

`[<package_name>:]`当引用的是自己应用中的资源时，可以省略。系统资源 android:

###引用样式属性
样式属性允许你使用当前主题提供的属性值。

这样可以不通过硬编码的方式的情况下，使得自定义UI元素的样式与当前主题提供的标准的变更匹配。

本质上就是：在当前的主题中，使用已经定义好的属性样式。

引用方式：

`?[<package_name>:][<resource_type>/]<resource_name>`

##访问平台资源
Android平台包含了许多标准的资源，如样式、主题和布局。
为了访问这些资源，package_name应该被限定为android