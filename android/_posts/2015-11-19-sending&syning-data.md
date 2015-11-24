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