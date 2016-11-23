---
layout: post
title: "APK signature scheme v2"
modified: 2016-11-23 13:48:46
excerpt: "Android 7.0引入的新的签名方式"
tags: [android,signature]
published: true
---

从 `Android 7.0` 开始引入了新的签名方案 `APK signature scheme v2`，此方案可以使应用安装更快，防止对 Apk 文件的非法修改提供更强大的保护。

默认情况下，在 Android Studio 2.2 开始，会默认使用v2 和传统的签名方案。

推荐使用此签名方式，但不强制。如果使用 v2 签名方式 app 不能正常编译，可以通过如下方式禁用此签名方式：

```
android { 
  ...     
  signingConfigs {
        release {
            storeFile file('myreleasekey.keystore')
            storePassword 'password'
            keyAlias 'MyReleaseKey'
            keyPassword 'password'
            // diable APK Signature Scheme v2
            v2SigningEnabled false
        }
    }
}

```

### 注意
如果通过 APK V2 签名 app，然后再修改，那么签名是无效的。因此应该在签名前使用诸如 zipalign 等工具，而不是之后。

还有就是多渠道打包，目前比较流行的2套多渠道打包脚本

- 在APK内注入${channel}.txt文件
- 在APK的zip info中写入 channel 信息

这两种方式都是在 app 签名后修改 APK 文件，进行打包的，因此导致 APK V2 签名失效，会导致在Android 7.0 设备上签名认证失败，安装不了的问题。现在的解决方法就是暂时禁用 APK V2 签名。