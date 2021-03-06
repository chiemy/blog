---
layout: post
title: "Material Design风格下Drawable的运用"
modified: 2015-11-11 14:51:03
excerpt: "本文介绍了在如何为Drawable着色，矢量图和矢量动画的运用，以及如何运用Palette类从图片中提取颜色"
tags: [android, material design, Palette, Vector drawable]
published: true
---
本文示例工程：[MaterialDesignDrawableDemo](https://github.com/chiemy/MaterialDesignDrawableDemo)

### 1.为Drawable着色

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/tint_drawable.png" width="300"><img>

#### 1.1 Android 5.0之前
在Android 5.0之前，我们可以使用v4支持库里的`DrawableCompat`类进行相关操作，示例如下：

{% highlight java %}
// 系统为了优化性能，相同的Drawble资源共享同一份拷贝，如果直接对drawable进行着色，则其他引用到该图片的敌方也会变色
// 为了打破这种共享，我们调用mutate()方法。但这并不是拷贝了一份相同的图片，内存开销比较小
Drawable drawable = DrawableCompat.wrap(iv.getBackground().mutate());
DrawableCompat.setTintList(drawable, ColorStateList.valueOf(Color.parseColor("#303F9F")));
iv.setBackground(drawable);
{% endhighlight %}

#### 1.2 Android 5.0之后
在Android 5.0之后，系统为我们提供了相应的Api，实现起来非常简单。

我们通过调用对象的`setTint()`方法对`BitmapDrawable`, `NinePatchDrawable` 或者 `VectorDrawable`进行着色处理，在xml文件中我们也可以通过相应的tint属性直接进行着色处理。

例如，在xml文件中，我们可以通过`android:drawableTint`属性，对`TextView`的诸如`android:drawableLeft`中的图片进行着色处理，可以通过`android:drawableTintMode`属性来指定着色的模式（默认为`SRC_ATOP`）。对背景图着色，可以用`android:backgroundTint`。

对`ImageView`的图片进行着色，我们使用xml文件中的`android:tint`属性，或者在代码中，我们可以使用ImageView的`setColorFilter(int color)`方法或者`getDrawable().setTint(color)`方法。

### 2.从图片中提取颜色

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/Palette.png" width="300"/>

在Android Support Library r21及之后的版本中，包含一个`Palette`类，通过它我们可以从一张图片中提取出图片的主要颜色，能提取的颜色包括：

- Vibrant
- Vibrant dark
- Vibrant light
- Muted
- Muted dark
- Muted light

#### 2.1使用：
添加依赖

{% highlight java %}
dependencies {
    ...
    compile 'com.android.support:palette-v7:21.0.0'
}
{% endhighlight %}

将被提取颜色的`Bitmap`传入`Palette`相应方法中

{% highlight java %}
// 同步，阻塞线程
 Palette p = Palette.from(bitmap).generate();
 // 提取颜色
 int vibrantColor = palette.getVibrantColor(defaultColor);
// 异步
 Palette.from(bitmap).generate(new PaletteAsyncListener() {
     public void onGenerated(Palette p) {
     		// 提取颜色
         int vibrantColor = p.getVibrantColor(defaultColor);
     }
 });
{% endhighlight %}

### 3.Vector Drawable（矢量图）的使用
在Android 5.0(API level 21)及以后，我们可以使用矢量图了。矢量图的好处在于，一个图形即可以适配任意分辨率，而又不会造成失真。

以下是官方文档中的一个例子，一个心型的矢量图：

{% highlight java %}
<!-- res/drawable/heart.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    <!-- intrinsic size of the drawable -->
    android:height="256dp"
    android:width="256dp"
    <!-- size of the virtual canvas -->
    android:viewportWidth="32"
    android:viewportHeight="32">

  <!-- draw a path -->
  <path android:fillColor="#8fff"
      android:pathData="M20.5,9.5
                        c-1.955,0,-3.83,1.268,-4.5,3
                        c-0.67,-1.732,-2.547,-3,-4.5,-3
                        C8.957,9.5,7,11.432,7,14
                        c0,3.53,3.793,6.257,9,11.5
                        c5.207,-5.242,9,-7.97,9,-11.5
                        C25,11.432,23.043,9.5,20.5,9.5z" />
</vector>
{% endhighlight %}

#### 3.1 属性解释

**`<vector>`下的属性**

- `android:viewportWidth / Height` 矢量图画布的大小，无单位，`<vector>`下所有成员的坐标都以此为参照。
- `android:width / height` 矢量图宽高（最好保持和画布一样的宽高比，否则会变形），会按照与`android:viewportWidth / Height`的比进行拉伸，但并不会失真。

**pathData中用到的SVG Path Data命令解释**

说明：

- 指令列：大写字母代表`绝对位置`，小写字母代表`相对位置`<br>
- 参数列：`+`表示可以有多个参数

<table summary="" class="" border="1" cellspacing="1">
<tr><th>指&nbsp;&nbsp;令</th><th>名称</th><th>参数</th><th>描述</th></tr>

<tr><td><strong>M</strong> or<br/>
         <strong>m</strong></td><td>moveto（移动到某点）</td><td>(x y)+</td><td>
          将画笔移动到某一点。
          <strong>M</strong> (大写) 表示绝对位置; <strong>m</strong> （小写)
          表示相对位置。如果后边有多个参数，视为lineto操作。 如果小写
          <strong>m</strong>作为path的起始命令，则第一个参数视为绝对位置（因为开始位置都是（0，0）点，所以等效），之后的参数视为相对位置。
        </td></tr>

<tr><td><strong>Z</strong> or<br/><strong>z</strong></td><td>closepath<br/></td><td>(none)</td><td>从当前的点滑一条直线指向起点，完成路径的闭合操作。不区分大小写</td></tr>

<tr><td><strong>L</strong> or<br/>
         <strong>l</strong></td><td>lineto（画线）</td><td>(x y)+</td><td>
从当前的点画一条线到给定的点(x, y)，然后以(x, y)点作为新的起点。
        <strong>L</strong> (大写)表示绝对位置的坐标; <strong>l</strong> (小写)表示相对位置的坐标。后边有多个参数时构成一条折线，最后的参数作为新的起点</td>
        
</tr><tr><td><strong>H</strong> or<br/>
         <strong>h</strong></td><td>horizontal lineto（画横线）</td><td>x+</td><td>从当前的点画一条横线。<strong>H</strong> (大写)表示绝对位置; <strong>h</strong>(小写)表示相对位置。 举例：如有点A(5, 0)， H10表示从A点画线到点（10，0）；h10表示从点A画线到（15，0）。可以有多个参数，但只以最后的参数为准，其余参数无效。</td></tr><tr>
        
<td><strong>V</strong> or<br/>
         <strong>v</strong></td><td>vertical lineto（画垂线）</td><td>y+</td><td>从当前的点画一条垂线。<strong>V</strong> (大写) 绝对位置; <strong>v</strong>
        (lowercase) 相对位置。举例：有点A(0, 5)，V10表示从A点画线到点（0，10）；v10表示从A点画线到点（0，15）</td></tr>
        
<tr><td><strong>C</strong> or<br/>
         <strong>c</strong></td><td>curveto(贝塞尔曲线)</td><td>(x1 y1 x2 y2 x y)+</td><td>Draws a cubic Bézier curve from the current
        point to (x,y) using (x1,y1) as the control point at the
        beginning of the curve and (x2,y2) as the control point at
        the end of the curve. <strong>C</strong> (uppercase)
        indicates that absolute coordinates will follow;
        <strong>c</strong> (lowercase) indicates that relative
        coordinates will follow. Multiple sets of coordinates may
        be specified to draw a polybézier. At the end of the
        command, the new current point becomes the final (x,y)
        coordinate pair used in the polybézier.</td>
        
</tr><tr><td><strong>S</strong> or<br/>
         <strong>s</strong></td><td>shorthand/smooth curveto</td><td>(x2 y2 x y)+</td><td>Draws a cubic Bézier curve from the current
        point to (x,y). The first control point is assumed to be
        the reflection of the second control point on the previous
        command relative to the current point. (If there is no
        previous command or if the previous command was not an C,
        c, S or s, assume the first control point is coincident
        with the current point.) (x2,y2) is the second control
        point (i.e., the control point at the end of the curve).
        <strong>S</strong> (uppercase) indicates that absolute
        coordinates will follow; <strong>s</strong> (lowercase)
        indicates that relative coordinates will follow. Multiple
        sets of coordinates may be specified to draw a
        polybézier. At the end of the command, the new
        current point becomes the final (x,y) coordinate pair used
        in the polybézier.</td></tr>

</table>

**用贝塞尔曲线画圆**

三次贝塞尔曲线非常适合用来绘制光滑连续曲线，因为只需要非常稀疏的数据集就能完整地绘制那些需要精确控制的曲线。

有些看上去很简单的曲线，例如圆，是无法用贝塞尔曲线或分段贝塞尔曲线精确描述的。可以用四段三次贝塞尔曲线模拟圆，每一段是一个四分之一圆。更一般地，我们可以用n段三次贝塞尔曲线模拟圆。

以下这个原型给出了一条一般化的样条曲线。经过拉伸、旋转、平移，四条这样的曲线就可以模拟任意圆和椭圆

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/magic%20number.gif"/>

网上查到求解x1和y1的方法（[相关文章](http://blog.csdn.net/cuixiping/article/details/6212047)），过程不是太明白，这里直接引用结果，x1=y1=0.551784。在使用时我们用此数值乘以半径，然后就能计算出各个点的坐标了。

#### 3.2 矢量图创建
一些简单的矢量图我们可以自己创建，但一些复杂图形我们自己来做就非常麻烦了。Android Studio内置了一些矢量图标可供我们使用，另外我们还可以导入其他的svg文件。

首先，我们在`drawable`文件夹上点击右键，选择`New > Vector Asset`

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/vector_asset.png" width="500">

选中`Material Icon`项，然后点击`Choose`按钮，这里为我们提供了许多图标，我们可以选择一个，然后点击`OK`，之后就会在`drawable`文件夹下生成一个包含`<vector>`节点的`.xml`文件，之后我们就可以引用了。

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/material_icon.png" width="500"/>


或者，我们可以导入其他SVG文件，选中`Local SVG File`项，然后选择我们本地的svg文件导入就可以了。由于默认的尺寸是24dp x 24dp，别忘了选中`Override default size from Material Design`，保持图片原有的宽高比，否则图片可能会变形。

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/local_svg_file.png" width="500"/>

貌似这个功能现在还有些bug，如上图，图片的边缘被截掉了一部分。

#### 3.3 矢量动画
矢量动画可以实现一些特殊的效果，使用它我们可以对图片中的某一个部分进行动画操作。如下图，我们可以对时钟图片的分针和时钟分别进行动画操作。

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/clock.gif" width="300"/>

下面以时钟动画举例，来看看矢量动画的使用。

**3.3.1 动画文件创建**

首先我们创建一个时钟的矢量图，代码如下。因为我们要单独对时钟和分针进行旋转操作，这里我们为时钟和分针分别套上一层`<group>`，设置名称和锚点。

{% highlight java %}
<!-- res/drawable/clock.xml -->
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="100dp"
    android:height="100dp"
    android:viewportHeight="24.0"
    android:viewportWidth="24.0">
    <!-- 表盘-->
    <path
        android:strokeWidth="2"
        android:strokeColor="#000000"
        android:pathData="M11.99,2
        C6.47,2 2,6.48 2,12
        s4.47,10 9.99,10
        C17.52,22 22,17.52 22,12
        S17.52,2 11.99,2z"
        />
    <!-- 时针-->
    <group
        android:name="hour"
        android:pivotX="12"
        android:pivotY="13"
        >
        <path
            android:strokeLineCap="round"
            android:strokeColor="#000000"
            android:strokeWidth="2"
            android:pathData="M12,8 v5"
            />
    </group>
    <!-- 分针 -->
    <group
        android:name="minute"
        android:rotation="135"
        android:pivotX="12"
        android:pivotY="13"
        >
        <path
            android:strokeLineCap="round"
            android:strokeColor="#000000"
            android:strokeWidth="2"
            android:pathData="M12,7 v6"
            />
    </group>
</vector>
{% endhighlight %}

然后，我们在`drawable`下创建一个以`<animated-vector>`为根节点的xml文件，然后为我们要操作的部分指定相应的动画。

{% highlight java %}
<!-- res/drawable/clock.xml -->
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/clock"
    >
    <!--为分针指定动画-->
    <target
        android:name="minute"
        android:animation="@anim/minute_rotation"
        />
    <!--为时钟指定动画-->
    <target
        android:name="hour"
        android:animation="@anim/hour_rotation"
        />
</animated-vector>
{% endhighlight %}

创建动画文件:

时钟旋转动画文件：

{% highlight java %}
<!-- res/anim/hour_rotation.xml -->
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="24000"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="360"
    android:repeatMode="restart"
    android:repeatCount="infinite"
    android:interpolator="@android:anim/linear_interpolator"
    />
{% endhighlight %}

分针旋转动画文件：

{% highlight java %}
<!-- res/anim/minute_rotation.xml -->
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:repeatMode="restart"
    android:repeatCount="infinite"
    android:duration="24000"
    android:propertyName="rotation"
    android:valueFrom="0"
    android:valueTo="4320"
    android:interpolator="@android:anim/linear_interpolator"
    />
{% endhighlight %}

**3.3.2 动画的播放**

图中的图片，我们是用`TextView`的`android:drawableLeft`引入的

{% highlight java %}
<!-- res/anim/minute_rotation.xml -->
<TextView
	android:id="@+id/tv_vector_anim1"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:drawableLeft="@drawable/animvectordrawable1"
  	android:layout_below="@+id/tv_vector"
   	android:text="Click to start vector drawable anim"
 	android:gravity="center_vertical"
  	/>
{% endhighlight %}

在代码中启动动画：

{% highlight java %}
TextView vectorAnimIv1 = (TextView) findViewById(R.id.tv_vector_anim1);
Drawable[] drawables1 = vectorAnimIv1.getCompoundDrawables();
for (int i = 0 ; i < drawables1.length ; i++){
	if (drawables1[i] instanceof Animatable){
		((Animatable)drawables1[i]).start();
	}
}
{% endhighlight %}
