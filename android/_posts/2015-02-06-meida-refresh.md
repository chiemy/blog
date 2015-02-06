---
layout: post
title: "Android媒体库更新问题研究"
modified: 2015-02-06 11:21:24
excerpt: "Android媒体库更新的机制，怎样通过代码方式让媒体库进行更新"
tags: [android, media]
comments: true
---

##1.媒体库何时更新？
在系统开机或sdcard卡被加载时，系统会自动扫描sdcard，将扫描到的如音频、图片等媒体文件保存到媒体数据库中，通过Android提供的相应的ContentProvider，我们可以获取这些媒体资源；

但如果我们在开机状态下，在sdcard内增加或删除一些媒体文件时，系统并不会自动扫描，因此媒体库不会更新（除非自行向媒体数据库中添加或删除）

##2.如何让媒体库更新？

###2.1.增加

####2.1.1.在Android 4.4之前我们可以通过以下方式，使系统扫描sdcard并更新媒体数据库：

	Intent intent = new Intent(Intent.ACTION_MEDIA_MOUNTED);
	intent.setData(Uri.parse("file://" + Environment.getExternalStorageDirectory().getAbsolutePath()));
	sendBroadcast(intent);
发送一个广播，让系统更新媒体数据库，也适用于删除媒体文件；

####2.1.2 在Android 4.4之后
但在Android 4.4中，限制了系统应用才有权限使用广播通知系统扫描SD卡，如果普通应用发送此广播将会出现异常：`Permission Denial: not allowed to send broadcast android.intent.action.MEDIA_MOUNTED`

可以采用如下解决方式：

**方式1：**

	MediaScannerConnection.scanFile(context, new String[] {“文件路径，包括文件名”}, null,null);

**方式2：**

	Intent intent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
	intent.setData(Uri.fromFile(new File(“文件路径，包括文件名”)));
	context.sendBroadcast(intent);

以上两种方式的局限性在于，只能指定扫描单个文件，不能指定文件夹，因此需要我们在下载文件完成时，使用上述方式更新媒体数据库；

###2.2 删除
在Android 4.4 之前，可以通过发送`Intent.ACTION_MEDIA_MOUNTED`广播的形式；

在Android 4.4之后，使用`ContentResolver`进行删除：

以删除音频文件为例：

通过路径删除

	context.getContentResolver().delete(
						MediaStore.Audio.Media.EXTERNAL_CONTENT_URI,
									MediaStore.Audio.Media.DATA+ " = '" + 音频文件全路径 + "'", null);