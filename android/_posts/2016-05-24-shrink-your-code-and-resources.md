---
layout: post
title: "压缩代码和资源"
modified: 2016-05-24 23:11:23
excerpt: "使用 Android Gradle 插件进行代码优化、混淆、压缩，及资源压缩"
tags: [android, gradle, ProGuard]
published: true
---


为了让我们的 APK 文件尽可能的小，我们可以压缩代码和资源。这篇文章将描述如何做到代码和资源的压缩，以及如何指定在构建过程中哪些内容应该保留，哪些内容应该丢弃。

`ProGuard` 可以用来进行代码压缩，它可以发现和移除打包的app中无用的类，变量，方法以及属性（包括支持库中的代码）。`ProGuard` 同时也会优化字节码，删除无用的代码说明，并且对保留的类，变量和方法使用短名称进行混淆。混淆的代码使得你的 APK 更难进行反编译的工作，当你的 app 使用了安全性较高的特性时（例如证书签名），这很有用处。

Android 的 Gradle 插件支持对资源的压缩，它会从打包的 app 中删除没有用到的资源（包括库工程中没有用到的资源）。这个是和代码压缩共同工作的，一旦无用的代码被删除后，那些与此关联的资源引用也可以被安全的删除了。

此文档中的一些功能基于：

- SDK Tools 25.0.10或更高
- Android Gradle 插件2.0.0或更高

##代码压缩

使用 `ProGuard` 进行代码压缩，首先在 `build.gradle` 文件中相关的 `build type` 下添加 `minifyEnabled true` 

注意，代码压缩将会延迟构建时间，所以，可能的话我们最好不要在 debug 时使用。然而，这对最终用于测试的 APK 很重要，因为，如果你没有充分的定制哪些代码需要保留，就可能引入 bug。

举例，下面的 `build.gradle` 中的代码片段为发布版本的构建开启了代码压缩。

```java
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile(‘proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
    ...
}
```

<br>

> 在使用 `Instant Run` （即刻运行）时，Android Studio 会关闭 `ProGuard`

除了 `minifyEnabled` 属性之外，`proguardFiles` 属性定义了 `ProGuard` 的规则：

- `getDefaultProguardFile(‘proguard-android.txt')` 方法，从 `Android SDK tools/proguard/` 文件夹下获取默认的 ProGuard 设置。

> Tips：为了更多的进行代码压缩，试下相同目录下的 `proguard-android-optimize.txt` 文件。它包含的了相同的 ProGuard 规则，但是它是从字节码层面进行优化的，这可以减少 APK 文件的大小，并且运行的更快。

- `proguard-rules.pro` 是你可以自定义 ProGuard 规则的文件，默认情况它处于模块根目录下。

为每个构建变种（build variant）添加特定的混淆规则，需要在相应的 `productFlavor` 下添加 `proguardFiles` 属性。例如，下面的 Gradle 文件中为 `flavor2` 添加了 `flavor2-rules.pro` ，现在 `flavor2` 可以使用所有的 ProGuard 规则，因为 `release` 代码块中的规则也同事被使用了：

```java
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                   'proguard-rules.pro'
        }
    }
    productFlavors {
        flavor1 {
        }
        flavor2 {
            proguardFile 'flavor2-rules.pro'
        }
    }
}
```
<br>

每次 build ，ProGuard 会输出如下文件：

`dump.txt`

描述 APK 文件下所有类的内部结构。

`mapping.txt`

原类、方法、变量的名称与混淆后的类、方法、变量名称的对应关系

`seeds.txt`

列举没有混淆的类和成员。

`usage.txt`

列举从 APK 中移除的代码。

这些文件处于 `<module-name>/build/outputs/mapping/release/` 目录下

### 定制哪些代码需要保持

在某些情形下，默认的 ProGuard 配置文件（`proguard-android.txt`）是足够的，ProGuard 会移除所有无用的代码。然而，在许多情形下，`ProGuard` 很难做出正确的分析，它可能会移除你需要的代码。例如，以下情况：

- 当你只从 `AndroidManifest.xml` 中引用一个类的时候。
- 当你的 app 从 JNI 调用方法的时候。
- 当你的 app 在运行时动态的操作代码的时候（比如反射）

当不正确的移除代码，在测试时会引发一些错误，但是我们也可以浏览下 `usage.txt` 文件检查下那些代码被移除了。

为修正错误，并且强制 ProGuard 保留特定的代码，需要在 ProGuard 配置文件中加入一行 `-keep` ，例如：

```
-keep public class MyClass
```

<br>

或者，我们可以为我们希望保留的代码夹 `@Keep` 注解。在类上添加 `@Keep` 注解将会保留整个类。在方法或属性上添加此注解，将会完整的保留方法和属性（包括名称）以及类名。注意，使用此注解需要引入 `Annotations Support Library`。

当使用 `-keep` 选项时，我们要考虑很多。更多关于定制配置文件的内容，见[ProGuard Manual. ](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html)，在[常见问题板块](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/troubleshooting.html)，列出了你可能会遇到的问题。

### 解读混淆的堆栈记录

在 ProGuard 压缩你的代码之后，阅读堆栈记录是很困难的事情，因为方法名称都被混淆了。幸运的是，ProGuard 每次运行时都会创建一个 `mapping.txt` 文件，来展示混淆后的类、方法、属性和原来的对应关系。

需要注意到的是，当你每使用 ProGuard 构建一个发布版的时候，`mapping.txt` 会被覆盖掉，因此，在每次你发布一个新的版本时，一定要留有一份拷贝。这样当用户给你反馈一个混淆后的错误信息时，根据 `mapping.txt` 的内容你可以定位该问题，并进行调试。

当在 Google Play 上发布你的 app 时，`mapping.txt` 文件也需要随版本一同上传。然后，Google Play 会对用户反馈的问题的堆栈跟踪信息进行反混淆，这样你就能 `Google Play Developer Console` 上看到这些问题。更多信息，见帮助中心关于 「[如何反混淆堆栈信息](https://support.google.com/googleplay/android-developer/answer/6295281)」的文章。

使用 `retrace` 脚本（Mac/Linux 上使用 `retrace.sh`，Windows 上使用 `retrace.bat`），我们可以自己将混淆后的堆栈跟踪信息还原成可读的信息。脚本位于 `<sdk-root>/tools/proguard/` 目录下。脚本接收 `mapping.txt` 文件和堆栈跟踪信息，会生成一个可读的堆栈跟踪信息，使用方法：

```
retrace.bat|retrace.sh [-verbose] mapping.txt [<stacktrace_file>]
```

<br>

例如：

```
retrace.bat -verbose mapping.txt obfuscated_trace.txt
```

<br>

如果不指定堆栈追踪信息所存储的文件，`retrace` 工具将会从标准输入中读入。

## 压缩资源

资源压缩只能和代码压缩一起工作，在代码压缩工具将所有无用代码移除后，资源压缩工具就能识别哪些资源还在继续被引用，这也同样适用于支持库中的代码和资源。

开启资源压缩，我们需要将 `build.gradle` 下的 `shrinkResources` 属性设置为 true。（和 `minifyEnabled` 一起）。例如：

```java
android {
    ...
    buildTypes {
        release {
            shrinkResources true
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
}
```

<br>


如果你还没有使用 `minifyEnabled` 创建你的 app，你需要在使用 `shrinkResources` 之前，做好相关配置。因为，在开始移除资源之前，你可能需要编辑你的 `proguard-rules.pro` 文件，来保留一些动态的创建或调用的类和方法。

> 注意：当前还不能移除 `values/` 文件夹下的资源（如，strings, dimensions, styles, colors），这是因为 Android Asset Packaging Tool (AAPT) 不允许 Gradle 插件为资源指定预定义的版本。详细信息见 [issue 70869](https://code.google.com/p/android/issues/detail?id=70869)

### 定制哪些资源需要保留

如果有特定的资源你希望保留或删除，那么，创建一个有 <resources> 标签的 XML 文件，然后，通过 `tools:keep` 属性保留指定资源，通过 `tools:discard` 移除指定的资源，两个属性都可以就收以逗号`,`作为分隔的资源列表。你可以用星号`*`作为通配符。

例如：

```xml
<?xml version=1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
    tools:discard="@layout/unused2" />
```

<br>

将此文件保存到项目资源文件夹下，如 `res/raw/keep.xml`， 它不会被打包到你的 APK 文件中。

貌似，指定哪些资源需要剔除看起来很蠢，直接删了不就行了？但当使用构建变种（build variant）时是很有用处的。例如，你也许会将所有的资源放在相同的项目目录下，然后，当你知道给定的资源在代码中会用到，但实际上特定的变种版本并不需要时，我们就可以为根据不同的构建变种创建不同的`keep.xml` 文件。

### 严格的引用检查

正常情况下，资源压缩工具能够准确的知道某个资源是否被使用。然而，当你在代码里使用 `Resources.getIdentifier()` 方法时，那意味着你的代码是基于静态的字符串来寻找资源的。当你这么做的时候，资源压缩工具，默认的会将所有匹配的资源当做可能会被用到的资源，并不可移除。

例如，下面的代码将会引起所有以 `img_` 作为前缀的资源标记为被用的：

```
String name = String.format("img_%1d", angle + 1);
res = getResources().getIdentifier(name, "drawable", getPackageName());
```

<br>

资源压缩工具也会查询你代码中的所有常量，以及 `res/raw/` 下的资源，寻找和 `file:///android_res/drawable//ic_plus_anim_016.png` 类似格式的资源 URLs，如果它发现一个字符串和此类似，或者其他类似的字符串要被用到 URLs 构造函数中，那么这些资源就不会被移除。

这种安全压缩模式是默认开启的，我们可以关闭此功能，让工具只保留我们明确用到的资源，我们需要设置 `keep.xml` 下的 `shrinkMode` 为 `strict`，像下边这样：

```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:shrinkMode="strict" />
```

<br>

如果你开启了 strict（严格的）压缩模式，并且你的代码中使用了静态字符串来引用资源，那么你必须手动的将这些资源添加到 `tools:keep` 下。

### 移除无用的可替换资源

Gradle 资源压缩工具只会移除没有被代码引用的资源，这意味着它不会移除为不同设备配置准备的可替换资源。如果必要的话，可以使用 Android Gradle 插件的 `resConfigs`属性来移除 app 不需要的可选资源文件。

例如，如果你的应用使用了一个包含语言资源的库（如AppCompat、Google Play Services），那么你的 APK 文件将会包含所有翻译后的语言字符，如果你只想保留你的 app 所正式支持的语言，可以使用 `resConfig` 属性来指定这些语言。任何未指定的语言资源将会被删除。

例如，下面的代码展示了如何限定只保留英语和法语的语言资源

```java
android {
    defaultConfig {
        ...
        resConfigs "en", "fr"
    }
}
```

<br>

定制 APK 文件包含哪些屏幕密度或 ABI 的资源，我们可以使用 [APK splits](http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits) 来为不同的设备构建 APK。

### 合并重复的资源

默认情况下，Gradle 会合并名称相同的资源，例如，位于不同文件夹下名称相同的图片。此行为不受 `shrinkResources` 属性控制，并且不能关闭，因为，这会避免有多个名称相同资源和你代码寻找的资源相匹配的错误。

资源合并只会发生在资源具有相同的名称、类型和修饰符。Gradle 会从中选择最合适的资源（按照下文的优先顺序），然后只将这一个资源传给 AAPT 

Gradle 将会在以下位置寻找重复资源：

- 与 main source set 相关的 main resources，通常位于 `src/main/res/` 下
- build type 和 build flavors 下的资源
- 依赖的库工程

Gradle 将会按照以下向下的优先顺序合并重复资源：

Dependencies → Main → Build flavor → Build type

越靠后的优先级越高，例如，如果 Main 和 Build flavor 下有重复的资源，那么 Gradle 会选择 Build flavor 下的资源。

如果，相同的资源出现在相同的 source set 下，Gradel 就不能合并他们了，会报一个资源合并的错误。当你在 `build.gradle` 中定义多个 `sourceSet` 属性时会出现这种问题，例如，`src/main/res/` 和 `src/main/res2/` 都包含相同的资源。

### 资源压缩常见问题

当你压缩资源时，Gradle 控制台会出现从 app 包中移除的资源的概要，例如：

```
:android:shrinkDebugResources
Removed unused resources: Binary resource data reduced from 2570KB to 1711KB: Removed 33%
:android:validateDebugSigning
```

<br>

Gradle 也会生成一个名为 `resources.txt` 的诊断文件，位于 `<module-name>/build/outputs/mapping/release/`
文件夹下，这个文件包含了相信的信息，例如，哪些资源引用了其他资源，哪些资源被使用，哪些被移除。

例如，我们想知道为什么 `@drawable/ic_plus_anim_016` 还在 apk 文件里，我们可以打开 `resources.txt` 文件，然后搜索那个文件名，你会发现它被其他资源引用到了:

```
16:25:48.005 [QUIET] [system.out] @drawable/add_schedule_fab_icon_anim : reachable=true
16:25:48.009 [QUIET] [system.out]     @drawable/ic_plus_anim_016
```

<br>

你现在需要知道，为什么 `@drawable/add_schedule_fab_icon_anim` 被用到了，然后向上搜索，你会看到它处在 `"The root reachable resources are:".` 下，这意味着在代码中使用到了 `add_schedule_fab_icon_anim`。

如果你没使用严格检查模式，当有字符串常量看起来像是要被用来通过动态编码的方式加载资源时，它会被标记为可用的。这种情况下，如果我们搜索这个资源名，会看到这样的内容：

```
10:32:50.590 [QUIET] [system.out] Marking drawable:ic_plus_anim_016:2130837506
    used because it format-string matches string pool constant ic_plus_anim_%1$d.
```

<br>

如果你看到这样的内容，并确定相关的资源并没有通过动态的方式进行加载，我们可以使用 `tools:discard` 属性通知构建系统将其移除，详见上文相关内容。
