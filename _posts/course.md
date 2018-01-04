---
layout: post
title: "教程"
modified: 2018-01-04 15:03:15
excerpt: ""
tags: [wechat]
published: false
---
# 1.下载抓包软件
[下载地址](http://ou3r6v4o4.bkt.clouddn.com/packet_capture.txt)
下载完后，把文件后缀.txt 改成 .apk，然后安装。

# 2.安装抓包软件
安装的时候在这个页面点第一选项
![](http://ou3r6v4o4.bkt.clouddn.com/screenshot.jpg)

出现对话框点确定

# 3.获取 Session id
###（1）点击这个按钮，找到微信，点击微信
![](http://ou3r6v4o4.bkt.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202018-01-04%20%E4%B8%8B%E5%8D%882.20.12.png)

###（2）：点击暂停按钮
![](http://ou3r6v4o4.bkt.clouddn.com/pause.png)

###（3）：杀掉微信，重新进入微信
###（4）：进入抓包软件，点击开始按钮
![](http://ou3r6v4o4.bkt.clouddn.com/start.png)
###（5）：进入微信跳一跳，等完全进入，不用开始游戏 
###（6）：再进入抓包软件，点击暂停按钮
###（7）：获取 session id，改分
点击第一条生成的记录，然后找到微信带有 SSL 标记的条目
![](http://ou3r6v4o4.bkt.clouddn.com/ssl.png)

点击进入，看是不是与下图中 1 处内容一致，如果不一致，返回继续找。
找到后，将下图中 2 对应处的 session_id 后引号内的内容拷贝出来，然后进入[微信跳一跳改分助手](http://java.zhaoxuyang.com/WxTyT/)，粘贴进去，输入分数，提交，就 OK 了。
最后杀掉微信，重新进跳一跳，看看分数变了没。
![](http://ou3r6v4o4.bkt.clouddn.com/WechatIMG55.jpeg)

