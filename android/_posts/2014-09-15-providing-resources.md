---
layout: post
title: "[API Guide]App Resources-可使用的资源"
modified: 2014-9-16 00:05:38
excerpt: "介绍可使用的资源文件类型，在哪里保存它们，以及怎样根据不同的设备配置创建可选择的资源"
tags: [android, Android Api, 翻译, App Resources]
comments: true
---

**Table 1.** Resource directories supported inside project `res/` directory.

<table class="table">
  <tr>
   <th scope="col">Directory</th>
   <th scope="col">Resource Type</th>
  </tr>

  <tr>
   <td><code>animator/</code></td>
   <td>Xml定义的<a href="http://developer.android.com/guide/topics/graphics/prop-animation.html">属性动画</a></td>
  </tr>

  <tr>
   <td><code>anim/</code></td>
   <td>Xml文件定义的<a href="http://developer.android.com/guide/topics/graphics/view-animation.html#tween-animation">补间动画</a>（属性动画也可以保存在这里，但优先选择animator/）</td>
  </tr>

  <tr>
   <td><code>color/</code></td>
   <td>颜色集合.See <a href="http://developer.android.com/guide/topics/resources/color-list-resource.html">Color State List Resource</a></td>
  </tr>

  <tr>
   <td><code>drawable/</code></td>
   <td><p>图片(.png, .9.png, .jpg, .gif)或者编辑成以下图片资源格式的Xml文件</p>
		<ul>
        <li>Bitmap files</li>
        <li>Nine-Patches (re-sizable bitmaps)</li>
        <li>State lists</li>
        <li>Shapes</li>
        <li>Animation drawables</li>
        <li>Other drawables</li>
      </ul>
		<p>See <a href="http://developer.android.com/guide/topics/resources/drawable-resource.html">Drawable Resources</a>.</p>
 	</td>
  </tr>

  <tr>
   <td><code>layout/</code></td>
   <td>布局文件</td>
  </tr>

  <tr>
    <td><code>menu/</code></td>
    <td>菜单（选项菜单，上下文菜单，子菜单）</td>
  </tr>

  <tr>
    <td><code>raw/</code></td>
    <td>保存任意文件的原始类型。通过Resources.getRawResource()指定的资源ID（R.raw.fileName）,得到InputStream
也可以将文件放在assets文件加下，这可以保存多个层级的文件，并且是使用AssetManager通过文件名直接访问的，这里的文件不生成资源ID.
</td>
  </tr>

  <tr>
    <td><code>values/</code></td>
    <td>Arrays.xml
		Colors.xml
Dimens.xml
Strings.xml
styles.xml</td>
  </tr>

  <tr>
    <td><code>xml/</code></td>
    <td>任意的xml文件，并且可以在运行期使用Resources.getXML().读取。
许多xml配置文件必须保存在这里，比如搜索配置。
</td>
  </tr>
</table>

>注意：不能在res/目录下存放文件，否则会引起编译错误

##提供可选的资源
几乎所有的应用都应该提供备选资源来支持特定的设备配置。比如，你应该为不同的屏幕密度提供可选的绘图资源，为不同的语言提供可选的字符串资源。Android将检测当前的设备配置，为你的应用加载适当的资源。

步骤：

1.在res/下创建文件夹规则如下：`<resources_name>-<config_qualifier>`

- `<resources_name>`为上表中文件的任意一种
- `<config_qualifier>`的命名如下表所示。可以有多个`<config_qualifier>`用“-”隔开

>注意：如果为同一类型文件夹追加限定符，那么必须按照表2的顺序排列，否则会被忽略

2.把资源保存各自的新路径下。资源的文件名必须与默认的资源文件名相当。

Android支持几种配置修饰语，你可以添加多个修饰语到同一个路径名，通过‘-’来分隔每一个修饰语。下表列举了可用的配置修饰语，考虑优先级，如果你为同一个资源路径使用了多个修饰语，你应该按照表中的列举的顺序添加。

<table class="table"> 
    <tr> 
     <th width="15%">Configuration</th> 
     <th width="15%">Qualifier Values</th> 
     <th width="70%">Description</th> 
    </tr> 
    <tr id="MccQualifier"> 
     <td>MCC and MNC</td> 
     <td>Examples:<br/> <code>mcc310</code><br/> <code>
       <nobr
        mcc310-mnc004
       </nobr</code><br/> <code>mcc208-mnc00</code><br/> etc. </td> 
     <td> 移动手机国家区号(MCC)，排在移动网络代码（MNC）之前。MNC来自设备的SIM卡。比如：mcc310表示美国，任何运营商；mcc310-mnc004表示美国Verizon；mcc208-mnc00表示法国Orange.<br/>

	如果设备使用无线电连接(GSM手机)，那么MCC和MNC的值来自于SIM卡。<br/>

	你也可以只使用MCC（例如：你的应用中包含了某个指定国家的合法资源）。如果你只是需要指定不同的语言，那么请使用语言和区域修饰语（见下文）。如果你决定使用MCC和MNC修饰语，你应该仔细的做一些测试以保证其正常工作。<br/>

	你也可以参与配置域mcc和mnc,他们分别指示了当前的移动手机国家区号和移动网络代码。 </td> 
    </tr> 
    <tr id="LocaleQualifier"> 
     <td>Language and region</td> 
     <td>Examples:<br/> <code>en</code><br/> <code>fr</code><br/> <code>en-rUS</code><br/> <code>fr-rFR</code><br/> <code>fr-rCA</code><br/> etc. </td> 
     <td> 语言由两个ISO 639-1语言代码组成，后面可以跟着两个由<ISO 3166-1-alpha-2>定义的区域代码字母(前面加小写字母“r”)。<br/>这些代码不区分大小写，前缀“r”是用来区分区域代码的部分。你不前单独使用区域代码。<br/>在应用程序的运行生命周期中，如果用户更改了他的系统语言设定，这些设置可以被更改。详见运行时变化处理- Handling Runtime Changes，了解其在运行时将怎么样影响你的应用程序。<br/>查看定位-Localization完整向导,了解怎样根据其他言语定位你的应用程序。<br/>同样可见locale配置域，其指示了当前的定位。 </td> 
    </tr> 
    <tr id="LayoutDirectionQualifier"> 
     <td>Layout Direction</td> 
     <td><code>ldrtl</code><br/> <code>ldltr</code><br/> </td> 
     <td>Ldrtl（layout-direction-right-to-left:布局方向从右向左） Ldltr（布局方向从左向右） 可以应用于layouts,drawables,values<br/> <strong>注意:</strong>如果想让你的应用可以使用right-to-left的布局，必须设置 supportsRtl为true，并且targetSdkVersion 至少17以上 </td> 
    </tr> 
    <tr id="SmallestScreenWidthQualifier"> 
     <td>smallestWidth</td> 
     <td><code>sw&lt;N&gt;dp</code><br/><br/> Examples:<br/> <code>sw320dp</code><br/> <code>sw600dp</code><br/> <code>sw720dp</code><br/> etc. </td> 
     <td> 屏幕的最小宽度是固定的，并不会随屏幕的方向改变而改变。 最小宽度的计算会考虑到系统界面的占用。比如，设备上沿着最小宽度轴线有些持久的UI元素，那么最小宽度要比实际的屏幕尺寸要小。因为，那些持久的UI元素,不属于你的应用， 
      <ul> 
       <li>320, for devices with screen configurations such as: 
        <ul> 
         <li>240x320 ldpi (QVGA handset)</li> 
         <li>320x480 mdpi (handset)</li> 
         <li>480x800 hdpi (high-density handset)</li> 
        </ul> </li> 
       <li>480, for screens such as 480x800 mdpi (tablet/handset).</li> 
       <li>600, for screens such as 600x1024 mdpi (7&quot; tablet).</li> 
       <li>720, for screens such as 720x1280 mdpi (10&quot; tablet).</li> 
      </ul> 当提供多种最小宽度的数值时，系统会挑选数值最接近的（向下兼容，不超过） Api13以上 android:requiresSmallestWidthDp </td> 
    </tr> 
    <tr id="ScreenWidthQualifier"> 
     <td>Available width</td> 
     <td><code>w&lt;N&gt;dp</code><br/><br/> Examples:<br/> <code>w720dp</code><br/> <code>w1024dp</code><br/> etc. </td> 
     <td> 指定最小的屏幕宽度，配置会随屏幕的方向改变。 当提供多种数值时，系统会挑选数值最接近的（向下兼容，不超过） 此数值不包含系统持久的UI元素，所以要比屏幕的实际数值小。<br/> Api13以上 screenWidthDp </td> 
    </tr> 
    <tr id="ScreenHeightQualifier"> 
     <td>Available height</td> 
     <td><code>h&lt;N&gt;dp</code><br/><br/> Examples:<br/> <code>h720dp</code><br/> <code>h1024dp</code><br/> etc. </td> 
     <td> 原理同上<br/> Api13之上<br/> screenHeightDp </td> 
    </tr> 
    <tr id="ScreenSizeQualifier"> 
     <td>Screen size</td> 
     <td> <code>small</code><br/> <code>normal</code><br/> <code>large</code><br/> <code>xlarge</code> </td> 
     <td> <strong>small</strong>:类似于低密度的QVGA屏幕的尺寸。小屏幕的最小布局尺寸约为320x426 dp。如QVGA低密度和VGA高密度。<br/><strong>normal</strong>:类似于中等密度的HVGA屏幕尺寸。一般屏幕的最小布局尺寸约340x470 dp。如WQVGA低密度，HVGA中等密度，WVGA高密度。<br/><strong>large:</strong>类似于中等密度的VGA屏幕尺寸。大屏幕的最小布局尺寸约480x640 dp。如中等密度的VGA和WVGA屏幕。 <br/><strong>xlarge:</strong>那些比传统中等密度HVGA更大的屏幕。加大屏幕的最小布局尺寸约720x960 dp。大多数情况下，加大屏幕的设备因为屏幕过大而无法放放口袋，一般为平板电脑类的设备。在API level 9中加入。<br/>  <strong>注意：</strong>如果使用的限定符超出了现在的屏幕，那么系统不会使用它们，并且会崩溃。比如，将资源文件标记为xlarge，但设备是normal-size的情况下。 </td> 
    </tr> 
    <tr id="ScreenAspectQualifier"> 
     <td>Screen aspect</td> 
     <td> <code>long</code><br/> <code>notlong</code> </td> 
     <td>long: 长屏幕, 如：WQVGA, WVGA, FWVGA<br/> notlong: 非长屏幕，如：QVGA, HVGA, and VGA <br/> 在API level 4中加入。<br/> 这个主要是基于屏幕的比例（“长”屏幕更宽一些）。和屏幕的方向无关。  </td> 
    </tr> 
    <tr id="OrientationQualifier"> 
     <td>Screen orientation</td> 
     <td> <code>port</code><br/> <code>land</code> 
      <!-- <br>
        <code>square</code>  --> </td> 
     <td> 
      <ul class="nolist"> 
       <li><code>port</code>: Device is in portrait orientation (vertical)</li> 
       <li><code>land</code>: Device is in landscape orientation (horizontal)</li> 
      </ul> <p>在应用程序的生命周期中这个配置会随着用户旋转屏幕而发生改变。详见<a href="http://chiemy.github.io/blog/handling-runtime-changes">处理运行时的变化</a>，了解其在运行时将怎么样影响你的应用程序。</p> </td> 
    </tr> 
    <tr id="UiModeQualifier"> 
     <td>UI mode</td> 
     <td> <code>car</code><br/> <code>desk</code><br/> <code>television<br/> <code>appliance</code> <code>watch</code> </code></td> 
     <td> 
      <ul class="nolist"> 
       <li><code>car</code>: Device is displaying in a car dock</li> 
       <li><code>desk</code>: Device is displaying in a desk dock</li> 
       <li><code>television</code>: Device is displaying on a television, providing a &quot;ten foot&quot; experience where its UI is on a large screen that the user is far away from, primarily oriented around DPAD or other non-pointer interaction</li> 
       <li><code>appliance</code>: Device is serving as an appliance, with no display</li> 
       <li><code>watch</code>: Device has a display and is worn on the wrist</li> 
      </ul> <p><em>Added in API level 8, television added in API 13, watch added in API 20.</em></p> <p>For information about how your app can respond when the device is inserted into or removed from a dock, read <a href="/training/monitoring-device-state/docking-monitoring.html">Determining and Monitoring the Docking State and Type</a>.</p> <p>This can change during the life of your application if the user places the device in a dock. You can enable or disable some of these modes using <code><a href="/reference/android/app/UiModeManager.html">UiModeManager</a></code>. See <a href="runtime-changes.html">Handling Runtime Changes</a> for information about how this affects your application during runtime.</p> </td> 
    </tr> 
    <tr id="NightQualifier"> 
     <td>Night mode</td> 
     <td> <code>night</code><br/> <code>notnight</code> </td> 
     <td> 会根据时间进行改变， 通过UiModeManager可以开启关闭此功能 </td> 
    </tr> 
    <tr id="DensityQualifier"> 
     <td>Screen pixel density (dpi)</td> 
     <td> <code>ldpi</code><br/> <code>mdpi</code><br/> <code>hdpi</code><br/> <code>xhdpi</code><br/> <code>xxhdpi</code><br/> <code>xxxhdpi</code><br/> <code>nodpi</code><br/> <code>tvdpi</code> </td> 
     <td> 120dpi.<br/> 160dpi.<br/> 240dpi.<br/> 320dpi<br/> 屏幕介于mdpi和hdpi，近似为213dpi<br/> 四个主要密度的比为：3:4:6:8，因此可按此比例设置图片的大小。 </td> 
    </tr> 
    <tr id="TouchscreenQualifier"> 
     <td>Touchscreen type</td> 
     <td> <code>notouch</code><br/> <code>finger</code> </td> 
     <td> notouch: 设备不支持触屏。<br/>finger: 设备有一个触摸屏。 </td> 
    </tr> 
    <tr id="KeyboardAvailQualifier"> 
     <td>Keyboard availability</td> 
     <td> <code>keysexposed</code><br/> <code>keyshidden</code><br/> <code>keyssoft</code> </td> 
     <td> 
      <ul class="nolist"> 
       <li><code>keysexposed</code>:设备有一个可用键盘。如果设备有一个软键盘可用，这个配置也可用，即使是物理键盘没有暴露给用户，甚至设置没有物理键盘。如果没有软键盘盘如是被禁用，那个这配置只有在有物理键盘被暴露时可用。</li> 
       <li><code>keyshidden</code>: 设备有一个物理键盘可用，但被隐藏了，且没有软键盘。</li> 
       <li><code>keyssoft</code>: 设备有一个软键盘可以，不管其是否可见</li> 
      </ul> <p>如果你提供了一个keysexposed资源，但没有keyssoft资源，系统将使用keysexposed资源，而不考虑键盘是否可见，如果系统有一个软键盘可用。 </p> <p>在应用程序的生命周期中这个配置会因用户打开一个物理键盘而发生改变。</p> </td> 
    </tr> 
    <tr id="ImeQualifier"> 
     <td>Primary text input method<br/> 首选文本输入方法 </td> 
     <td> <code>nokeys</code><br/> <code>qwerty</code><br/> <code>12key</code> </td> 
     <td> 
      <ul class="nolist"> 
       <li><code>nokeys</code>: 设备没有物理键用作文本输入。</li> 
       <li><code>qwerty</code>: 设备有一个物理的 qwerty 键盘，不管其是否对用户可见。</li> 
       <li><code>12key</code>: 设备有一个物理的 12-key 键盘，不管其是否对用户可见。 </li> 
      </ul> <p>Also see the <code><a href="/reference/android/content/res/Configuration.html#keyboard">keyboard</a></code> configuration field, which indicates the primary text input method available.</p> </td> 
    </tr> 
    <tr id="NavAvailQualifier"> 
     <td>Navigation key availability</td> 
     <td> <code>navexposed</code><br/> <code>navhidden</code> </td> 
     <td> 导航键可用<br/> 不可用 </td> 
    </tr> 
    <tr id="NavigationQualifier"> 
     <td>Primary non-touch navigation method</td> 
     <td> <code>nonav</code><br/> <code>dpad</code><br/> <code>trackball</code><br/> <code>wheel</code> </td> 
     <td> 无导航<br/> 方向间<br/> 轨迹球<br/> 方向轮<br/> </td> 
    </tr> 
    <tr id="VersionQualifier"> 
     <td>Platform Version (API level)</td> 
     <td>Examples:<br/> <code>v3</code><br/> <code>v4</code><br/> <code>v7</code><br/> etc.</td> 
     <td>设备支持的Api版本 </td> 
    </tr> 
  </table>


###规则

- 可以追加多个限定符，用破折号分开
- 必须按照表二的顺序添加
- 资源文件不能嵌套，只能有一层目录结构
- 值是不区分大小写的，为避免在不区分大小写的系统中出现问题，资源编译器会将目录名转换为小写。名字中出现的任意大写字母只是为了增加可读性。

###创建别名资源
当你想在不同的系统配置下使用一份资源的时候，你不必把资源放在不同的资源文件夹下。你可以创建一个别名资源。

>注意：animation，menu，raw和其他未指定的处于xml/文件夹下的资源不支持此功能。

假如你有一个名为icon.png的应用图标，需要为不同的区域提供不同的图标。但是，有两个区域使用的是同一张图标。这时你可能会将图标拷贝一份放到不同的文件夹下。然而，这不是必须的。我们可以将这两个区域用到的图标命名为icom_ca.png（任意不为icon.png的名称），保存在res/drawable/文件夹下。然后创建一个使用<bitmap>元素引用icon_ca.png图标的icon.xml文件，分别置于对应的两个区域的文件夹下。这样就可以只保存一个png文件，节省空间。

####Drawable
为图片创建别名资源，如下所示

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
 	android:src="@drawable/icon_ca" />
{% endhighlight %}

####Layout
为布局创建别名，

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<merge>
 	<include layout="@layout/main_ltr"/>
</merge>
{% endhighlight %}

####字符串和其他基本资源

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
 	<string name="hello">Hello</string>
 	<string name="hi">@string/hello</string>
</resources>
{% endhighlight %}

##提供最好的设备兼容资源
在提供不同适配资源的时候，最好保留一个默认的，比如values/，layout/，这样即使没有合适的资源匹配当前的系统，也不会出现系统崩溃的情况。

提供默认资源是很重要的，不仅是因为你的应用可能会跑在你没有预料到的配置上，还因为新版的Android有时会提出新的限定符，而旧版本不支持。例如，你的应用提供两种drawable资源，一种是night，一种notnight（Api8加入的夜间模式功能），但应用却运行在了Api版本为4的机器上，那么程序就会崩溃。我们可以将drawable作为我们的默认资源(notnight),drawable-night作为夜间模式资源。

###安卓系统是怎么寻找最佳资源的

<font color="00FF00">
drawable/<br/>
drawable-en/<br/>
drawable-fr-rCA/<br/>
drawable-en-port/<br/>
drawable-en-notouch-12key/<br/>
drawable-port-ldpi/<br/>
drawable-port-notouch-12key/<br/>
</font>

假设设备的配置如下：

<font color="00FF00">
Locale = en-GB <br/>
Screen orientation = port <br/>
Screen pixel density = hdpi <br/>
Touchscreen type = notouch <br/>
Primary text input method = 12key<br/>
</font>

通过系统配置和资源文件的比较，安卓系统从drawable-en-port下获取图片。

系统运用如下逻辑来选择资源：

1.排除不符合设备配置的资源

首先排除不符合en区域的。

<font color="00FF00">
drawable/<br/>
drawable-en/<br/>
<S>drawable-fr-rCA/ </S><br/>
drawable-en-port/<br/>
drawable-en-notouch-12key/<br/>
drawable-port-ldpi/<br/>
drawable-port-notouch-12key/<br/>
</font>

例外：屏幕的像素密度不会被排除

2.从表二中，按照顺序向下（从MMC开始）。

3.那些资源目录包含此限定符？

- 如果不包含，就回到第二步，继续查找
- 如果有，就到第四步

4.排除不包含此限定符的资源文件，

<font color="00FF00">
<s>drawable/</s><br/>
drawable-en/<br/>
drawable-en-port/<br/>
drawable-en-notouch-12key/<br/>
<s>drawable-port-ldpi/</s><br/>
<s>drawable-port-notouch-12key/</s>
</font>

5.继续2、3、4步骤，直到只剩下一个文件夹。

<font color="00FF00">
<s>drawable-en/</s><br/>
drawable-en-port/<br/>
<s>drawable-en-notouch-12key/</s>
</font>

<figure>
	<a href="http://developer.android.com/images/resources/res-selection-flowchart.png"><img src="http://developer.android.com/images/resources/res-selection-flowchart.png"></a>
	<figcaption><a href="http://developer.android.com/images/resources/res-selection-flowchart.png" title="资源匹配过程">资源匹配过程</a></figcaption>
</figure>

对于屏幕尺寸的限定符，如果没有合适的资源，那么会向下兼容。例如，大尺寸的屏幕在没有合适资源的时候会使用normal-size的资源，但是，如果设备是normal-size的，而只有large-size的资源，系统就不会选择使用这个资源，而会崩溃。

>注意：优先级要远比匹配的上的数量重要，比如上边的例子，虽然drawable-port-notouch-12key 资源文件有三个限定符符合当前系统配置，但drawable-en只匹配了一个限定符，但是en限定符有更高的优先级，所以，drawable-port-notouch-12key文件先于drawable-en文件被排除。

###要知道的问题
Android 1.5 and 1.6对资源文件的要求是完全匹配。也就是说，不会出现向下兼容的情况，必须严格的匹配才行


