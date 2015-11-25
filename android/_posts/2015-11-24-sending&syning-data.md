---
layout: post
title: "Android Wear发送及同步数据"
modified: 2015-11-24 09:58:57
excerpt: "数据发送及同步"
tags: [android, andorid wear]
published: false
---
Android Wear数据层API，是Google Service的一部分，为我们的手机应用和Android wear应用提供了相互通信的通道。API中包含了一系列的数据对象，可以帮助我们发送和同步数据。

- Data Items:

　　`DataItems`类似于共享队列的方式，不需要支持其他节点的Id，只需要把数据放到队列中即可。

- Messages

　　`MessageApi`可以发送消息，利于远程过程调用（RPC），例如通过手表控制手机应用的播放器。类似于广播的方式，需要知道其他的节点的Id才可以对给其发送消息。

- Channel

	我们可以使用`ChannelApi`来传输较大的data items，例如音乐、电影文件，从手机端到手表端。有以下几个优点：	
	
	1. 在传输大文件上比`MessageApi`可靠
	2. 可传输数据流
	
- Asset

	`Asset`对象用来传输二进制的数据，例如图片。把assets附加在data items上，系统会自动管理数据传输，通过缓存大的assets来避免重新传输节省蓝牙带宽。
	
- WearableListenerService（for service）

	继承`WearableListenerService`，可以监听重要的数据层事件。系统对其生命周期进行管理，当需要发送data items或mssages时绑定该服务，在不需要时解除绑定。
	
- DataListener (for foreground activities)

	在activity中实现`DataListener`，在activity处于前台时监听重要的数据层事件。当我们仅需要在用户使用我们的app时对数据进行监听，我们使用`DataListener`，而不使用`WearableListenerService`。
	

