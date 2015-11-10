---
layout: post
title: "如何创建、运行可穿戴设备App?"
modified: 2015-11-10 13:17:51
excerpt: "本文将介绍如何创建、运行可穿戴设备的App"
tags: [android, android wear]
published: true
---

##本文依赖的测试环境
**操作系统**：Mac OS X 10.11 (15A284)

**开发环境**：Android Studio 1.4

**可穿戴设备**：Moto 360二代(Android Wear 1.3.2265291, Android系统 5.1.1)

**手持设备**：LG Nexus 5(Android 6.0)


##1.升级SDK
在创建可穿戴设备app前，需要确定SDK版本：

- SDK tools版本23.0.0及以上
- 需要4.4W.2(API 20)及以上SDK

##2.配置Android wear设备
可以用虚拟机进行测试，在这里就不做介绍了，可以到官网看如何配置虚拟机。[Set Up an Android Wear Emulator or Device](http://developer.android.com/intl/zh-cn/training/wearables/apps/creating.html#SetupEmulator)。

下面主要讲一下Moto 360配置步骤：

###2.1 下载Android Wear手机应用，与手表配对。
这步说起来貌似有些多余，没有Android Wear是不能激活设备的，就更别提调试了。

本人是在乐商店下载的国行版应用，也可以在Google Play下载。

> 国行版应用和Google Play版Android Wear应用功能和使用上是有差别的，具体差别问下度娘。

###2.2 在手表上开启**开发者选项**
和Android手机开启开发者选项类似，进入**设置/Settings > 关于/About**，点击**版本号/Build number**七次，就会开启开发者选项了。

![](../version_number.png)


###2.3 开启手表的调试模式
回到设置中，点击进入**开发者选项/Developer options**，开启ADB调试，由于Moto 360不支持USB调试，需要通过蓝牙进行调试，那么我们同时将蓝牙调试模式开启，如图：

![](../debug.png)

###2.4 连接手表
进入手机端Android Wear应用，进入**设置/Settings**，开启**通过蓝牙调试/Debugging over Bluetooth.**，此时主机状态为断开状态，目标状态为已连接。如图所示：

![](../bt_debug01.png)

然后就是与主机进行连接，手机连接电脑，在电脑终端输入以下命令：

	adb forward tcp:4444 localabstract:/adb-hub
	adb connect localhost:4444

此时，手表端会显示“是否允许调试?”的提示，点击“确定”或“始终允许”就可以了：

![](../bt_debug03.png)

好啦，到这里基本配置就结束了，接下来我们就可以使用Moto 360进行调试了。

##3.创建工程
下面，我们创建一个工程，看一下在Moto 360上运行的效果吧！

首先，打开Android Studio，点击“Start a new Android Studio project”，这个步骤与创建普通工程没有区别，填写好工程信息之后，我们进入下一个界面，这步比较关键，选中"Wear"项，如图：

![](../create_wear_project.png)

接下来的过程比创建普通工程多了一个步骤，在为手机端应用添加Activity之后，有个为Android Wear应用添加Activity的步骤。这个步骤我们先不仔细探究，一路点"Next"，我有些迫不及待了！

等待工程创建好后，我们看下目录结构，发现创建了两个Module，一个名为"mobile"，另一个名为"wear"，通过名称应该就能理解是什么意思了。

![](../wear_project.png)

下面到了最激动人心的时刻了，让我们运行一下来看看效果！在运行按钮前的选择框，我们选择"wear"这个module，然后点击运行，如果之前的配置正确，手表处于连接状态的话，对话框里会显示手表的运行选项，我们双击手表项（图中的Motorola Moto 360）运行。

![运行对话框](../run_dialog.png)

受到蓝牙传输速度的影响，可能要等上一段时间。稍后，我们就会在手表上看到熟悉的"Hello World"界面了！

![Moto 360demo运行界面](../hello_world.png)


##4.Tips
###4.1 卸载应用
执行命令如下命令即可：
	
	adb -s localhost:4444 uninstall 包名
	
卸载的速度还是很快的，就是向手表发了一个指令。

###4.2 手表屏幕截屏
手机端的Android Wear自带截图功能，但是发送截屏指令后虽然提示截屏成功，但是并不能获取这个文件，不知道是什么问题。

还有一种方式，就是通过Android Studio的截屏工具截屏。在"Android Monitor"标签下，我们选择我们的手表，然后点击“照相机”图标就可以截屏了。

![Moto 360demo运行界面](../watch_captrue.png)


##5.示例工程
[GitHub：AndroidWearDemo](https://github.com/chiemy/AndroidWearDemo)

