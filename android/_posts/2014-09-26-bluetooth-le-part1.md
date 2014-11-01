---
layout: post
type: android
title: "低功耗蓝牙（1）"
modified: 2014-9-26 10:16:11
excerpt: "RecyclerView的使用方法"
tags: [android, bluetooth ,翻译]
comments: true
published : true
---
原文地址：[Bluetooth LE – Part 1](http://blog.stylingandroid.com/bluetooth-le-part-1/)

在写这篇文章的时候，Google刚刚发布Android Wear，同时摩托罗拉也发布了Moto 360智能手表。Wear APIs还是相当基础的（很好的文档），并且还会有更多东西，所以我不打算写一个关于它教程（至少现在）。一个有趣的事情是，Moto 360支持Android 4.3及之后的设备，这可能暗示着Moto 360将会支持低功耗蓝牙（BLE/Bluetooth LE/Bluetooth Low Energy）。低功耗蓝牙将成为一个重要的功能，不止限于可穿戴技术，还有物联网（loT-Internet of Things）设备。在此系列的文章中，我们将看看在Android上低功耗蓝牙的使用。

蓝牙在20世纪90年代中期就已经出现，并已经成为短距离对等网络通信的标准。它有一个缺点就是功耗很大，这在手机上就成为了一个问题，在可穿戴设备上的问题更大，因为它的电池更小。蓝牙的配对过程只需要一次，但对用户来说体验很痛苦。

低功耗蓝牙，在Bluetooth 4.0(有时称作Bluetooth Smart)说明中作为一个部分，解决了这些具体问题。随着电池寿命越来越重要，许多厂商声称可以工作数月甚至几年的时间（因为厂商都是基于较好的环境下测试的，不包括真实的使用场景，所以我对此表示怀疑）。正如已经提到的，Google已经为Android 4.3(API 18)添加了BLE支持。

对与那些已经对蓝牙很熟悉的人来说，BLE有一定的难度，因为它和原来相比有很大的不同。让我们来看看都有那些变化。

首先就是配对过程。传统的蓝牙中，配对任务更多的由用户负责，但在BLE中则主要有开发者负责。在用户角度来说这是件好事，因为它使得配对过程更简单明了。

另一个主要的区别是通讯方面，在传统的蓝牙开发中有几个选项。都是基于和标准的网络sockets类似的socket结构。本质上我们的数据是通过sockets进行传输的，传输中两个设备都明确数据流的格式。BLE则采用不同的方法，并基于一些属性。一个属性是两个设备间分享的数据实质上是原子性(atomic)的(i.e. 一个integer或string)。属性既可以用来代表数据，也可以对感应器进行控制。例如，在一个心率监测器中，一个属性可能携带着当前的心率值(数据)，另一个可能包含着心率值更新的频率的设置(行为)。

在我们继续深入探讨之前，需要制定一些规矩。通常我们会将传感器(诸如，心率监测器、温度传感器)连接到主机上(诸如手机，电脑)。在我们的例子中，我们的主机可以在从一个属性读取数据的同时，通过给另一属性重新赋值的方式来控制传感器。There is actually a little more to it than that because a host can also register to be notified when an attribute changes, but we’ll cover that in due course.

在这一系列的文章中，我将使用的设备是[Texas Instruments SensorTag](http://www.ti.com/tool/cc2541dk-sensor#3),它是个BLE、多传感器的开发套装，价格25美元，包含温度、红外、湿度、气压计、磁力仪、加速度计、陀螺仪。在这系列文章里，我们将创建一个应用，可以展示从SensorTag中获取的环境温度和湿度。

抱歉文章中没有代码，但我觉得在深入细节问题(nitty gritty)之前，建立一个背景是很重要的。我保证下篇文章你就会见到代码了

