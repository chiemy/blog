---
layout: post
title: "压缩代码和资源补充"
modified: 2016-10-18 11:27:21
excerpt: "对代码和资源压缩的一些补充"
tags: [android]
published: true
---

## 分析APK大小工具

### Analyz APK
Android Studio 2.2自带的`Analyz APK` 工具，可以查看 APK 各组成部分所占用的空间大小

使用 `Build -> Analyz APK` 选择 APK 文件，或者双击 APK 文件。

<img src="../../images/android/anylyz_apk.png" width="600"/>

### ClassShark 
ClassShark是一款查看Android执行文件的工具，可以查看诸如类的数量、方法数、dex数量等重要信息。

<img src="https://github.com/borisf/classyshark-user-guide/blob/master/images/5%20ClassesDexData.png?raw=true"
 width=600/>

使用方法见：[ClassShark Github](https://github.com/google/android-classyshark)


## 使用APK Splits

### 按屏幕密度拆分

```
android {
  ...
  splits {
    density {
      enable true
      exclude "ldpi", "tvdpi", "xxxhdpi"
      compatibleScreens 'small', 'normal', 'large', 'xlarge'
    }
  }
```

- enable： 启用屏幕密度拆分机制
- exclude： 默认情况下所有屏幕密度都包括在内，你可以移除一些密度。
- include： 表示要包括哪些屏幕密度
- reset()： 重置屏幕密度列表为只包含一个空字符串 （这能够实现，在与include一起使用时可以表示使用哪一个屏幕密度，而不是要忽略哪一些屏幕密度）
- compatibleScreens：表示兼容屏幕的列表。这将会注入到manifest中匹配的 节点。这个设置是可选的。

### 使用多版本的APK

Multiple APK Support是一个在Google Play，可以发布不同的应用程序，分别针对不同的设备配置特征。每个APK是一个完整的、独立的应用程序版本，但他们分享在Google Play相同的应用程序清单，必须共享相同的包名和与签名。Google Play 会自动给你匹配相应的APK，这样我们的APK 就可以是分不同版本构建需要资源文件，从而减小APK的大小。

通过发布有多个APK，我们可以：

- 支持不同OpenGL的APK
- 支持不同的屏幕尺寸和密度的APK
- 支持不同的设备功能的APK
- 支持不同的平台版本的APK
- 支持不同的CPU架构，每个apk（如ARM、x86，MIPS等)的APK

更多相关信息请参考[https://developer.android.com/google/play/publishing/multiple-apks.html#Concepts](https://developer.android.com/google/play/publishing/multiple-apks.html#Concepts)
