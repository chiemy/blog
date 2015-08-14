---
layout: post
title: "监控电量和充电状态"
modified: 2015-08-14 13:50:37
excerpt: ""
tags: [android, training, performance]
published: true
---

我们可以通过动态注册广播，来监控一些手机状态的变化。

同时，还有静态注册的方式（在应用的manifest文件中，为每个你想监控的状态注册`BroadcastReceiver`，然后在相应的接收器的实现里对相应的状态进行处理。）

这种方式不好的地方在于，监听的状态越多，你的应用唤醒设备就会越频繁。为了解决这个问题，我们可以在运行时，对广播进行开启或关闭。

### 对状态改变接收器进行开关，提高效率

我们可以使用`PackageManager`对定义在manifest中的任意组件进行状态切换，包括广播接收器，代码如下：

{%highlight java%}

ComponentName receiver = new ComponentName(context, myReceiver.class);

PackageManager pm = context.getPackageManager();

pm.setComponentEnabledSetting(receiver,

        PackageManager.COMPONENT_ENABLED_STATE_ENABLED,

        PackageManager.DONT_KILL_APP)

{%endhighlight%}