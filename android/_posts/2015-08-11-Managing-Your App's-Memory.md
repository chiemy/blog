---
layout: post
title: "内存管理建议"
modified: 2015-08-11 22:03:09
excerpt: "本文介绍了一些内存优化与管理的建议和注意点"
tags: [android, memory]
comments: true
---

>本文内容参考自Android官网[Managing Your App's Memory](http://developer.android.com/training/articles/memory.html)，大部分为翻译内容。

RAM是非常宝贵的资源，尤其是在手机操作系统上。尽管Android下的虚拟机有垃圾回收机制，但内存管理的问题也不容忽视。

>虚拟机会回收没有被引用的内存空间，因此要注意App中的一些全局变量造成的内存泄露问题。

### 内存管理建议

### 1.慎用Service

当开启一个服务时，系统会为该服务保留一个进程，这个代价是非常高的。

如果你的应用中有一些处理后台工作的服务，在完成后台工作后，要及时的将后台服务关闭。更好的方式是使用`IntentService`，它会在处理完启动它的意图对象后自动的结束掉。（`IntentService`的[使用](https://developer.android.com/training/run-background-service/index.html)）

在一个服务不需要后仍然保持它的运行，是内存管理严重失误之一。一直保持服务的运行，不仅会存在内存不足的风险，而且在用户知道问题后，很可能会将你的程序卸载掉。

### 2.在UI隐藏时释放内存

当用户跳转到其他应用，且你的应用的UI不可见时，你应当释放一些只有你的UI才用到的一些资源。这样可以显著提高系统缓存能力，以及提升用户体验。

为了接收退出应用UI的事件，需要重写`Activity`的`onTrimMemory()`方法，其中` TRIM_MEMORY_UI_HIDDEN`就表示UI已经不可见，可以释放资源了。

注意，仅当应用所有的UI组件都不可见时，才会接收到`TRIM_MEMORY_UI_HIDDEN`事件。注意和`onStop()`的区别，`onStop()`方法是`Activity`不可见时被调用的，经常出现于界面跳转时。

### 3.在内存吃紧时释放内存

在整个App的生命周期中，如果设备的内存低到一定程度时，`onTrimMemory()`也会被调用。

以下是可能受到的一些参数：

-`TRIM_MEMORY_RUNNING_MODERATE`

你的App在运行，且系统不考虑将你的应用杀死。但是系统的内存资源已经相对紧张了，系统会杀死一些

LRU缓存中的进程。

-`TRIM_MEMORY_RUNNING_LOW`

你的App在运行，且系统不考虑将你的应用杀死。但是系统的内存资源已经非常低了，你需要释放一些资源来保证你的应用的性能。

-`TRIM_MEMORY_RUNNING_CRITICAL`

你的App在运行，且系统已经清除了大部分LRU缓存中的进程。你应该释放所有不是很重要的资源。如果系统没有足够的内存空间，它会开始清理所有LRU缓存，以及杀死一些进程，如正在运行的Service。

同时，如果你的进程被缓存后，你将接受到如下参数：

-`TRIM_MEMORY_BACKGROUND`

系统内存较低，且你的进程处在LRU列表的开头。尽管你的应用进程被杀死的风险不大，但它已经开始从LRU缓存中进行清理工作了。因此，你应当释放一些比较容易恢复的资源，尽可能让你的进程保持在列表中。

-`TRIM_MEMORY_MODERATE`

系统内存较低，且你的进程处于LRU列表的中部，如果内存进一步吃紧的话，你的应用进程有可能会被杀死。

-`TRIM_MEMORY_COMPLETE`

系统运行内存较低，且如果系统不能恢复一些内存的话，你的进程将处于被杀进程的队列中。你应当释放所有的对恢复你的应用状态无关紧要的资源。

因为`onTrimMemory() `方法在`API 14`之后才可用，在旧版本上你可以使用`onLowMemory()`方法，它类似于`TRIM_MEMORY_COMPLETE`事件。

>注：虽然LRU的机制是从底部开始清理的（最近最少使用），但同时进程所占的内存也会在考虑范围内，所以越少的内存消耗，对常驻LRU列表越有利，并且可以快速回到你的应用。

### 4.检查可用内存

不同的Android驱动的设备有不同的可用内存，因此会对app有不同的内存限制。你可以通过`getMemoryClass() `来估计一下你的应用的可用内存空间。如果你的应用尝试获取超过此可用内存空间，那么就会接收到`OOM`错误。

在极特殊的情况下，你可以通过将`<application>`标签下的`largeHeap`属性设置为`true`，来请求一个较大的空间。此时，你可以通过`getLargeMemoryClass()`来估算一下可用空间。

然而，仅一些真正需要大内存的应用（例如图片处理软件）建议请求大内存。**永远不要因为你的应用运行出现了OOM错误，想要快速解决此问题而请求大的内存空间**，只有当你确切的知道你的所有的内存都分配到哪了，且为什么一定要保留这些内存（也就是说，有充分的理由要保留这些内存，而选择不释放）的时候，你才能请求大内存空间。即使你确信你的应用有充分的理由来请求大的空间，也要尽可能的避免。使用额外的内存空间会损害用户体验，因为垃圾回收要销毁较长时间，同时进行其他操作时，系统性能就会非常低。

另外，请求的大内存空间在不同的设备上也会不一样，当运行在比较小的内存的设备上时，大的内存空间值，可能只跟大部分设备的正常内存值是一样的。因此，尽管你请求了大的内存空间，你也要通过`getMemoryClass()`来检查内存的使用情况，不要超过内存限制。

### 5.避免Bitmap导致的内存浪费

当加载位图的时候，仅在内存中保留适合当前分辨率的大小的图片，如果原图分辨率较高的话，应当进行压缩处理。

>注：在Android 2.3.x(Api 10)及以下，无论分辨率多大，bitmap对象的大少是一致的（实际的像素数据保存在native memeory中）。这使得对bitmap内存分配的调试变得很难，因为大部分heap分析工具，不会分析native分配。从Android 3.0之后，bitmap的像素数据是分配在你的应用的Dalvik heap中的，提升了垃圾回收和调试性。因此在一些老的设备上遇到的位图的问题，可以在Android 3.0以上进行调试。

更多建议见：[Managing Bitmap Memory.](http://developer.android.com/training/displaying-bitmaps/manage-memory.html)

### 6.使用优化的数据容器

利用Android框架中的一些容器，例如`SparseArray`，`SparseBooleanArray`以及`LongSparseArray`。`HashMap`的实现逻辑效率是非常低下的，因为每个键值的保存都需要一个单独的entry对象。另外，由于`SparseArray`避免了系统的自动的封箱和开箱，所以`SparseArray`效果更高。

### 7.知晓内存开销

了解你使用的语言及libraries的开销，设计应用的从始至终，都要时刻谨记。表面上看起来无害的东西可能对开销影响很大。以下的这些点，可能颠覆你之前的认识。例如

-枚举是常量占用内存的两倍，尽量避免在android中使用枚举。（糟糕！我就喜欢用枚举）
-任何类（包括匿名内部类）需要约500字节的空间。
-每个类会有约12-16字节的内存开销。
-`HashMap`每存入一个键值对，需要为entry对象分配额外的32字节的空间。

这些字节一点点的累积，如果app设计中存在着许多类和对象，那么问题会更突出。

### 8.谨慎使用抽象

开发者经常会认为，运用抽象是好的编程实践，因为他会提高代码的灵活性和稳定性。然而，抽象带来了一个严重的消耗：通常要执行很多的代码，需要更多的时间和RAM空间将代码mapped到内存中。因此，如果抽象没有给你带来显著的好处，应该避免使用他们。

>OMG!这叫我如何权衡？

### 9.使用`nano protobufs`序列化数据

`Protocol buffers`（[Google官网](https://developers.google.com/protocol-buffers/docs/overview)、[百度百科](http://baike.baidu.com/link?url=9zaBM9M2q637E21PWQUz4epr0L-r2rGHKvEodmmE5ZJPsi04HoAeKSlsTpJ1uJf3VAvzdPrshHh7SNsI8v7ne_)）是Google设计的一种用于序列化结构性数据的，可扩展的机器语言，它更快、更小、更简单。

如果，你要使用此protobufs，在客户端首选nono protobufs。一般的protobufs会生成非常冗长的代码，这会给你的应用带来很多问题，如：RAM使用的增长，签名apk文件大小的增加，缓慢的执行速度。

更多内容见[protobuf readme](https://android.googlesource.com/platform/external/protobuf/+/master/java/README.txt)

### 10.避免使用依赖注入框架

一些依赖注入框架（如[Guice](https://github.com/google/guice)、[Roboguice](https://github.com/roboguice/roboguice)）是很诱人的，它们可以简化你的代码。然而，他们会启动许多进程来来扫描代码中的注释。这些映射（mapped pages）会被分配到干净的内存（clean memory）中,在很长时间后才会被清除掉。

>还好我没用过

### 11.谨慎使用外部库

许多外部库不是为手机环境写的，如果应用再手机平台上可能会很低效。因此在使用前你要肩负起针对手机平台进行优化的重担，要考虑的大小、RAM占用情况等问题。

即使是专门运行在Android平台上库也可能存在潜在的风险，因为你不知道库里都做了些什么事情。

同时要避免为了使用库中众多特性中的一两个特性，而使用库的陷阱。你应该不想引入大量你用不到的代码。如果你找不到能够很大程度上适合你需求的实现方法，那么你最好自己尝试着去实现。

### 12.使用`ProGuard`剔除多余代码

[`ProGuard`](http://developer.android.com/tools/help/proguard.html)工具可以通过删除无用代码，对类、文件、方法进行重命名的方式，对你的代码进行压缩、优化以及混淆。使用`ProGuard`可以使你的代码更坚实，以及减少RAM占用。

### 13.对最终Apk文件进行`zipalign`处理

如果你使用构建系统生成了APK文件，那么你必须运行[zipalign](http://developer.android.com/tools/help/zipalign.html)对其进行处理优化。如果不做此处理，那么你的app将会需要更多的RAM空间，because things like resources can no longer be mmapped from the APK.

>Google Play Store 不允许ape不经过zipalign处理

### 14.RAM使用分析

当你的项目相对稳定的时候，需要分析一下各生命周期阶段你的App对RAM的使用情况。关于如何分析，参见[Investigating Your RAM Usage](http://developer.android.com/tools/debugging/debugging-memory.html)

### 15.使用多进程

如果合适的话，有一项可以将你的app各个组件分成不同进程的先进技术，可以帮助你更好的管理内存。这项技术的运用要格外小心，并且大部分应用并不适合运用此技术，因为如果运用不得当的话，不会减少RAM的开支，反而会增加。它对那些前后台同时运行任务，并且可以单独对这些任务进行管理的应用比较有用。

可以通过为组件声明[`android:process`](http://developer.android.com/guide/topics/manifest/service-element.html#proc) 属性，来为组件指定特定进程的目的。例如，通过为一个Service声明一个名为`background`的进程，就可以指定该Service在应用的主进程以外的进程运行。

{% highlight xml%}

``` 
<service android:name=".PlaybackService"
         android:process=":background" />
```

{% endhighlight%}

进程的名称应该以`:`开头，来确保该进程是私有的。

在你决定使用这项技术之前，需要知晓如下内容：

-一个空进程会占用大概1.4MB的RAM空间，如果再执行一些任务，那么占用的空间会迅速上升。
-如果你要将你的应用分成多个进程，那么应该只有一个进程来处理UI（经测试，如果一个进程只展示一个显示一点文字的Activity，那么RAM占用会到4MB左右）。
-你的代码要保持尽可能的简洁，应为各进程共同的实现部分所带来的不必要的RAM开销，都会被各个进程共有。例如，如果你使用了枚举（上文提到不建议使用枚举，会造成额外的RAM开销），会造成所有引用该枚举类型的进程都有额外的RAM开销（即造成呈进程倍数的增长）。

