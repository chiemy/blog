---
layout: post
title: "[Android Tv]Android Tv应用布局"
modified: 2015-12-29 17:53:34
excerpt: "Android Tv应用布局"
tags: [android, andorid tv]
published: ture
---

电视屏幕比其他的Android 设备屏幕都要大，一般都是从3米之外的地方观看，这样的屏幕不会像小屏幕一样提供丰富的细节和颜色。这些因素促使我们，在开发电视应用时要创建有用的、有趣的用户体验。

## 1 主题的运用

### 1.1 Leanback主题
`v17 leanback library`支持库提供了标准的电视端Activity的主题，叫`Theme.Leanback`。这个主题为电视应用提供了一致的用户视觉体验。

{% highlight java%}
<activity
  android:name="com.example.android.TvActivity"
  android:label="@string/app_name"
  android:theme="@style/Theme.Leanback">
{% endhighlight %}

### 1.2 NoTitleBar主题
TitleBar在手机和平板应用上是标准的接口元素，但并不适用于电视应用。如果应用中不使用`v17 leanback library`中的主题，那我们要确保使用不带TitleBar的主题。

{% highlight java%}
<application>
  ...
  <activity
    android:name="com.example.android.TvActivity"
    android:label="@string/app_name"
    android:theme="@android:style/Theme.NoTitleBar">
    ...
  </activity>
</application>
{% endhighlight %}

## 2 基础布局的创建
为了确保电视应用布局的高效，我们要遵循一些基础原则：

- 创建横向布局，因为电视总是横向显示的。
- 将导航放到布局的左侧或右侧，纵向空间要留給要展示的内容
- 分区域创建UI，使用Fragment，使用`GridView`而不使用`ListView`，达到更充分利用横向空间的目的。
- 使用如`RelativeLayout`或`LinearLayout`的ViewGroup控件来排列视图。
- 避免界面杂乱，视图间保留足够的间距

## 3 OverScan
由于电视的演进，及对全屏图像的期望，电视应用布局有着特殊的要求。由于这些原因，为了让画面全屏显示，电视机可能会将应用布局的边缘截取掉，这种现象被成为`OverScan`。

为了保证我们应用界面中的布局元素的可见性，我们需要在布局的左右边缘加上48dp的边距，上下边缘加上27dp的边距。

{% highlight java%}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
   android:layout_width="match_parent"
   android:layout_height="match_parent"
   >

   <!-- Screen elements that can render outside the overscan safe area go here -->

   <!-- Nested RelativeLayout with overscan-safe margin -->
   <RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
       android:layout_width="match_parent"
       android:layout_height="match_parent"
       android:layout_marginTop="27dp"
       android:layout_marginBottom="27dp"
       android:layout_marginLeft="48dp"
       android:layout_marginRight="48dp">

      <!-- Screen elements that need to be within the overscan safe area go here -->

   </RelativeLayout>
</RelativeLayout>
{% endhighlight %}

> 注意：不要为`v17 leanback`下的类进行此设置，如`BrowseFragment`或者相关的控件，因为他们已经做过此设置。

相关资料：

- [Baidu 过扫描](http://baike.baidu.com/link?-url=0ozFdd0lXqJojygSxoZithYUEt-1ZO__n-v96vS2VLgu4TBCvfrDDhY1OS4dNDMKv4EL1nQP9OuGDOJ1sn8ivq)
- [Wiki Overscan](https://en.wikipedia.org/wiki/Overscan)
- [重显率](http://baike.baidu.com/link?url=DVhimQrfZA1VUwO3HRrLqVwWY1x5bCSydTHRP6RPixhGIBQYota2JJJ80Q-zJMH0WaCiCz6kVsn6TbslAd4piK)

## 4 创建可用的文字和控制
电视应用中的文字及控制项，应该很容易的让用户在远距离看到及导航，下面是一些建议：

- 将文字切割为段落，让用户很容易阅读。
- 在深色背景上使用浅色字体，这样更容易阅读。
- 避免使用轻便（lightweight）的字体、非常狭窄及边框很粗的字体。使用简单的无衬线及抗锯齿字体增加可读性。
- 使用标准的字体尺寸，`android:textAppearance="?android:attr/textAppearanceMedium"`
- 保证在3米以外能够清楚的看到屏幕上的视图。最后的方式就是使用相对布局，使用`dp`作为单位。

更多屏幕适配内容见[Supporting Multiple Screens](https://developer.android.com/guide/practices/screens_support.html)

## 5 布局资源管理
普通高清电视的分辨率为720，1080i和1080p。我们应该以1920x1080的屏幕尺寸为基准进行适配。

## 6 应避免的布局方式
有以下几种创建布局的方式是我们需要避免的

- 重用手机或平板的布局
- ActionBar
- ViewPager -在手机和电脑上是横好的体验，但不要在电视上尝试。

更多创建适用于电视应用的布局的信息，请查看[Tv Design guide](https://developer.android.com/intl/zh-cn/design/tv/index.html)




