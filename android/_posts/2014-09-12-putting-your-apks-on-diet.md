---
layout: post
title: "让APK文件更小"
modified: 2014-9-11 15:38:18
excerpt: "缩减Apk文件大小的一些建议"
tags: [android, tips, 翻译]
comments: true
---

原文地址：[Putting Your APKs on Diet](http://cyrilmottier.com/2014/08/26/putting-your-apks-on-diet/)

Android最初版本的APK文件只有2MB左右的大小，而现在的应用变得越来越大，动辄就是10几20MB。用户体验和开发者经验的积累是造成APK文件越来越大的直接原因，还有以下一些原因：

- dpi种类的增加（[l|m|tv|h|x|xx|xxx]dpi)）
- 安卓平台、开发工具、libraries生态系统的演进
- 用户对于高品质UI的期望的不断增长
- ……

在Play-store上发布轻量型的应用，是每个开发者在开发应用时就该关注的。为什么呢？首先，因为它和简单、可维护、不过时的代码是统一的。
其次，开发者更愿意遵守Play Store对Apk文件50MB的大小限制，而不愿意去处理下载扩展文件。最后，因为我们生活在一个有限制的世界中：有限的带宽，有限的磁盘空间等等。APK文件越小，下载越快，安装越快，挫败感越小，更重要的是获得更好的评价。

许多情况下，APK的文件大小的增长是为了满足用户的需求和期望而被迫进行的。然而，我确信大小的增长已经快过用户期望的增长了。事实上，Play Store中的一些应用的大小要比它们应该或必须有的尺寸大上2倍或者更多。在这篇文章中，我将讨论一些能够减少Apk文件大小的技术/准则，这将使你的同事和用户更加开心。

##APK文件的构成

在看一些非常有用的减少Apk文件大小的方法之前，我们必须了解一下APK文件的构成。简单的说来，Apk文件就是一个保存多个文件的压缩文件。解压之后，可以看到下面的目录结构：

{% highlight html %}
/assets
/lib
  /armeabi
  /armeabi-v7a
  /x86
  /mips
/META-INF
  MANIFEST.MF
  CERT.RSA
  CERT.SF
/res
AndroidManifest.xml
classes.dex
resources.arsc
{% endhighlight %}

上边的一些文件有些是我们比较熟悉的，他们通常与开发过程中的目录结构对应，如：`/assets`,`/libs`,`/res`,`AndroidManifest.xml`。其它的一些看起来比较陌生，`classes.dex`，包含java代码编译的dex版本，`resources.arsc`包括一些预编译资源，如二进制的XML文件，drawable等。

因为Apk是一个简单的档案文件，所以它有两个大小：解压缩后的大小和未解压的大小。两个都很重要，本文着重于解压后的大小。事实上，我们可以认为为解压大小和Apk文件是成比例的：Apk文件越小，未解压文件也越小。

##缩减APK文件大小
通过几种技术可以对APK文件的大小进行缩减。因为每个应用都是不同的，因此没有绝对的规则。然而，我们可以从APK的三个组成部分着手：

- Jave代码
- resources/assets
- 本机代码

以下的一些技巧就是从以上部分着手，来减少APK文件的大小的。

###保持代码的整洁
没错，保持代码的整洁是减少APK文件大小的第一步。除去无用的Libraries，持续保持整洁。

保持代码库的整洁，在项目的开头往往非常容易。项目越久，难度就越大。那些拥有较长历史的项目，不得不经常处理无用的代码片段。幸运的是，有许多开发工具可以帮助我们做这些整理工作。

###使用混淆器
混淆器是在编译时对你的代码进行混淆、优化、压缩的强有力工具。它能够减少Apk大小的主要特性是tree-shaking，它会遍历你所有的代码路径，检测出那些没用的代码片段，所有这些多余的代码片段，都会从最终的APK文件中被剔除。混淆器同时也会对你的文件、类、接口进行重命名，保证代码尽可能的轻量级。

混淆器是非常有用和有效的，但职责越大，影响也越大。许多程序猿很讨厌混淆器，因为它破坏了app的结构关系，而且还得去配置混淆器，告诉它什么文件、类该处理或者不该处理。

###广泛使用Lint工具
混淆器作用于Java层面，不幸的是，它不能作用于资源层面。因此，如果一张在`res/drawable`中的图片未被使用，那么它只会把R文件中的引用剔除，可不会帮你把文件删除的。 

Lint是一个静态代码的分析工具，能够帮你检测到未使用的资源文件。通过它的分析功能我们可以很安全的删除这些文件。

Lint会分析资源（i.e.在/res目录下的文件），但不会分析`/assets`下的文件。`assets`文件是通过文件名而不是通过Java或者XML的引用来访问的。因此，Lint不能检测出此目录下的文件是否被使用。这就需要我们程序猿自己来保证`assets`文件的清洁啦。

>关于Lint工具的使用，请见[Lint工具的使用](http://developer.android.com/tools/debugging/improving-w-lint.html)

###适合的resources
Android支持非常多的设备。事实上，Android能忽略设备配置(如屏幕分辨率、大小、形状)为设备提供支持。例如，Android 4.4框架能够支持：ldpi, mdpi, tvdpi, hdpi, xhdpi, xxhdpi 和 xxxhdpi的分辨率的设备。虽然它提供了这么多分辨率，但是你的应用有必要都支持吗？

如果你的应用只是为一少部分人开发的，那么别害怕做出取舍。我个人的应用就只支持 hdpi, xhdpi ，xxhdpi的分辨率的设备，这并不会在其他分辨率的设备上出现什么问题，因为Android会通过缩放资源的方式自动的匹配合适的资源。

我这个观点（只支持 hdpi, xhdpi ，xxhdpi）背后的原则非常简单。

首先，它涵盖了80%的用户。

其次，xxxhdpi是为了适应未来而存在的，而不是现在。

最后，我根本不关心糟糕的低分辨率的设备。

同样，在`drawable-nodpi`中保存单张图片也可以节约你的空间。如果你不认为拉伸图片很粗暴，或者这张图片在整个应用的日常使用过程中很少用到，你就可以这样做。

###最小化resources配置
Andorid开发经常依赖一些外部的库，比如，Android support　Library、Google Play Service、Facebook SDK等等。所有这些库会自带一些对你的应用毫无用处的资源。例如，Google Play Service自带了对一些语言的翻译，然而这些语言可能你的应用根本就不支持。它同时也捆绑了我不愿支持的`mdpi`资源。

从 Android Gradle Plugin 0.7开始，通过`resConfig` 和 `resConfigs`以及默认的配置选项，你可修改构建系统的配置信息。DSL阻止打包工具打包那些不符合app的资源配置。

{% highlight java %}
defaultConfig {
 	// ...
 	resConfigs "en", "de", "fr", "it"
 	resConfigs "nodpi", "hdpi", "xhdpi", "xxhdpi", "xxxhdpi"
}
{% endhighlight %}

###压缩图像
打包工具自带[图片无损压缩算法](http://developer.android.com/guide/topics/resources/drawable-resource.html#Bitmap),例如，一张不超过256色的真彩色的PNG图片可能会通过调色板被转换成8位的PNG。这可能减少你的资源的大小。在Google上可以搜索到一些PNG压缩处理的工具，比如`pngquant`, `ImageAlpha` 和 `ImageOptim`。

另外,Android平台上一种特有的图片格式`.9`,也可以节约空间。

###限制架构数量
Android通常是Java代码，但有些情况下需要依赖native-code，你可以像优化resources一样优化它。在当前的Android生态系统下， armabi 以及 x86 架构已经足够了。这有一篇关于如何缩减native-libraries的文章非常不错。[文章链接](http://blog.algolia.com/android-ndk-how-to-reduce-libs-size/)

###尽可能多的复用
在手机上开发应用，可能最重要的就是学会“复用”了。在`ListView`或`RecyclerView`中，“复用”使得滚动过程很流畅。同时“复用”也能够帮助你减少APK文件的大小。例如，在新的Android L版本中使用`android:tint`以及`android:tintMode`，或者使用适用于所有版本的`ColorFilter`，能够为`assets`重新配色。

如果应用中需要多张图片，它们的内容相同，但角度不同，就不需要保存多张。我们可以通过xml文件来实现对图片的旋转，并达到节省空间的目的。

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
 	android:drawable="@drawable/ic_arrow_expand"
 	android:fromDegrees="180"
 	android:pivotX="50%"
 	android:pivotY="50%"
 	android:toDegrees="180" />
{% endhighlight %}

###更进一步？
以上的技术都是从应用/支持库开发者的角度进行讨论的。如果我们能够在Android链条上有完全的控制，是否能更进一步呢？我猜我们可以，但是那会涉及到服务，或者更明确的说是Play-store角度。例如，我们可以想象Play-Store的打包系统只捆绑在目标设备上需要的natice-libraries.

我们可以想象，只打包目标设备需要的配置。不幸的是，那会完全破坏最重要的功能：热插拔。的确，Android始终被设计成能够处理动态配置变化（语言，屏幕方向）。删除不与目标设备屏幕密度兼容的资源会带来很大的好处。不幸的是，Android应用可以动态的处理屏幕密度的改变。我们可以反对这种功能，我们没必要把精力放在去适配不同分辨率的设备上，就算一种设备的适配就够我们受的了（方向，最小宽度，等等）

服务端Apk打包看似强大，但是也非常危险，因为最终到达用户手里的应用会与我们发给Play-store的有很大不同。

###总结
设计是关于如何从一系列的限制中解放出来，APK文件的大小就是这限制中的一个。让应用的其他方面突出，就别怕在应用的某个方面做出妥协。例如，如果降低UI渲染的质量能够减少APK文件的大小并且使得应用更小巧，那么别犹豫。99%的用户不会注意到UI质量的下降，他们更关心应用是否轻量级与流畅。终究，评判一个应用是从整体，而不是局部。