---
layout: post
title: "Android Wear设备App打包"
modified: 2015-11-10 13:17:51
excerpt: "本文将介绍如何将Android Wear应用与手机应用一同打包"
tags: [android, android wear]
published: true
---
在上篇文章中([上篇文章](http://chiemy.com/android/android-wear-create/))，我们介绍了如何创建Android Wear工程，并且在Moto 360上进行调试。之前我们都是通过Android Studio直接将测试工程安装到Android Wear的，那么如何实现安装手机应用的同时，自动将Android Wear应用推送到Moto 360上呢？

##1.用Android Studio打包
步骤：

1.将Android wear module的manifest文件中添加的权限，同样声明到手机module的manifest文件中。例如，Android wear module使用到了VIBRATE权限，那么也要将此权限添加到手机module的manifest文件中。

2.确保Android wear module和手机module有相同的包名及版本号。

3.在手机module的build.gradle文件下添加如下依赖：

	dependencies {
   		compile 'com.google.android.gms:play-services:5.0.+@aar'
   		compile 'com.android.support:support-v4:20.0.+''
  		wearApp project(':wearable')
	}
	
> 注：wearApp project(':wearable')中的wearable改为对应的module名称
	
4.点击**Build > Generate Signed APK...**，在接下来的界面，使用keystore对应用进行签名。Android Studio会导出一个嵌入Android Wear应用的手机端应用。

##2.安装
接下来，我们将签名的手机应用安装到手机上，一切正常的话手机会向手表同步与之相应的Android wear应用。

但要注意的是，Moto 360必须要用国际版Android wear激活，否则无法将应用同步到手表端。

这里多吐槽下Android wear国行版：

- 不能同步第三方应用到手表
- 乐商店官方提供的应用少的可怜，在官方下了几个应用同样不能同步，这是几个意思？
- 应用商店应用少不承认也就算了，弄个“加载中……”来欺骗用户？我更是醉了！

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/fuck.png" alt="Drawing" width="300" />




	

