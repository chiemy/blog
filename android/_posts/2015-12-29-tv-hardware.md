---
layout: post
title: "[Android Tv]Android Tv硬件控制"
modified: 2015-12-29 10:24:30
excerpt: "对Android Tv硬件的使用"
tags: [android, andorid tv]
published: false
---

##1 设备检测
如果创建了可在Tv和其他设备上能够同时操作的app，我们怎么知道应用运行在什么设备上呢？我们可使用`UiModeManager.getCurrentModeType()`方法，如下：

{% highlight java %}
public static final String TAG = "DeviceTypeRuntimeCheck";
UiModeManager uiModeManager = (UiModeManager) getSystemService(UI_MODE_SERVICE);
if (uiModeManager.getCurrentModeType() == Configuration.UI_MODE_TYPE_TELEVISION) {
    Log.d(TAG, "Running on a TV Device")
} else {
    Log.d(TAG, "Running on a non-TV Device")
}
{% endhightlight %}

##2 处理不支持的硬件属性

###2.1 电视设备不支持的硬件属性

<table>
  <tr>
    <th>Hardware</th>
    <th>Android feature descriptor</th>
  </tr>
  <tr>
    <td>Touchscreen</td>
    <td><code>android.hardware.touchscreen</code></td>
  </tr>
  <tr>
    <td>Touchscreen emulator</td>
    <td><code>android.hardware.faketouch</code></td>
  </tr>
  <tr>
    <td>Telephony</td>
    <td><code>android.hardware.telephony</code></td>
  </tr>
  <tr>
    <td>Camera</td>
    <td><code>android.hardware.camera</code></td>
  </tr>
  <tr>
    <td>Near Field Communications (NFC)</td>
    <td><code>android.hardware.nfc</code></td>
  </tr>
  <tr>
    <td>GPS</td>
    <td><code>android.hardware.location.gps</code></td>
  </tr>
  <tr>
    <td>Microphone <sup><a href="#cont-mic">[1]</a></sup></td>
    <td><code>android.hardware.microphone</code></td>
  </tr>
  <tr>
    <td>Sensors</td>
    <td><code>android.hardware.sensor</code></td>
  </tr>
</table>

> [1] Some TV controllers have a microphone, which is not the same as the microphone hardware feature described here. The controller microphone is fully supported.

更多属性、子属性见[Features Reference](https://developer.android.com/guide/topics/manifest/uses-feature-element.html#features-reference)


###2.2 电视设备上的硬件使用
如果你的应用使用了一些在电视上不可用的硬件特性，但不用这些特性也可操作，那么需要在manifest文件中声明不需要此特性。即使我们的应用在其他非电视设备上使用了这些特性，我们也需要为电视设备将其声明为不可用，代码如下：

{% highlight java %}
<uses-feature android:name="android.hardware.touchscreen"
        android:required="false"/>
<uses-feature android:name="android.hardware.faketouch"
        android:required="false"/>
<uses-feature android:name="android.hardware.telephony"
        android:required="false"/>
<uses-feature android:name="android.hardware.camera"
        android:required="false"/>
<uses-feature android:name="android.hardware.nfc"
        android:required="false"/>
<uses-feature android:name="android.hardware.location.gps"
        android:required="false"/>
<uses-feature android:name="android.hardware.microphone"
        android:required="false"/>
<uses-feature android:name="android.hardware.sensor"
        android:required="false"/>
{% endhighlight %}

> 注：一些特性下还包含子特性，如相机下的`android.hardware.camera.front`，我们同时要办证这些特性也要标记为`required="false"`

> <font color="#ff0000">注意</font>：如果标记一些电视上不支持的硬件特性为`true`，那么应用将不能安装或不会出现在主界面中。

###2.3 一些依赖硬件属性的权限
manifest中对一些权限的请求是依赖一些硬件特性的，当我们声明的权限依赖了不支持的硬件特性时，也要注意将相应属性标记为`false`。以下是一些例子，更多对应关系见[uses-feature guide](https://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions-features)：

<table>
  <tr>
    <th>Permission</th>
    <th>Implied hardware feature</th>
  </tr>
  <tr>
    <td><code><a href="/reference/android/Manifest.permission.html#RECORD_AUDIO">RECORD_AUDIO</a></code></td>
    <td><code>android.hardware.microphone</code></td>
  </tr>
  <tr>
    <td><code><a href="/reference/android/Manifest.permission.html#CAMERA">CAMERA</a></code></td>
    <td><code>android.hardware.camera</code> <em>and</em> <br>
      <code>android.hardware.camera.autofocus</code></td>
  </tr>
  <tr>
    <td><code><a href="/reference/android/Manifest.permission.html#ACCESS_COARSE_LOCATION">ACCESS_COARSE_LOCATION</a></code></td>
    <td><code>android.hardware.location</code> <em>and</em> <br>
      <code>android.hardware.location.network</code></td>
  </tr>
  <tr>
    <td><code><a href="/reference/android/Manifest.permission.html#ACCESS_FINE_LOCATION">ACCESS_FINE_LOCATION</a></code></td>
    <td><code>android.hardware.location</code> <em>and</em> <br>
      <code>android.hardware.location.gps</code></td>
  </tr>
</table>


#to be continued……

