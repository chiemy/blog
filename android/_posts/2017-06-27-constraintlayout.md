---
layout: post
title: "ConstraintLayout 的使用"
modified: 2017-06-27 19:35:17
excerpt: "使用 ConstraintLayout 创建更加偏平化的布局"
tags: [android]
published: true
---

<br>
ConstraintLayout （约束布局）在不嵌套布局的情况下，创建比较复杂的布局。它和 RelativeLayout 很相似，控件的位置都是跟兄弟控件和父控件相关联的，但它比 RelativeLayout 更加强大，在 Android Studio 布局编辑中更容易使用。

在 Android Studio 布局编辑器的可视化工具下，可以通过拖拽的方式来创建 ConstraintLayout 布局。

## 约束布局的添加

按照如下步骤添加：

1. 点击 **Tools > Android > SDK Manager**
2. 点击 **SDK Tools** 标签
3. 展开 **Support Repository** 项，然后选中 **ConstraintLayout for Android** 和 **Solver for ConstraintLayout**。选中 **Show Package Details** 记住版本，接下来要用到。
4. 点击 **OK**
5. 然后在应用的 `build.gradle` 下添加依赖

```
dependencies {
    compile 'com.android.support.constraint:constraint-layout:1.0.0'
}
```


6.最后点击 **Sync Project with Gradle Files.**

### 将已存在的布局转换为约束布局

通过以下方式，可以将一个已经存在的布局转换为约束布局：

<a><img src="http://7o4zgd.com1.z0.glb.clouddn.com/constraintlayout01.png" align="middle" width="500"></a>

1. 选择 **Design** 视图
2. 在 **Component Tree** 窗口下，右击布局，点击**Convert layout to ConstraintLayout**

### 创建新的约束布局

通过以下方式，创建约束布局文件：

1. 在 **Project** 窗口下，点击模块文件夹，然后选择 **File > New > XML > Layout XML**
2. 输入文件名，以**android.support.constraint.ConstraintLayout**作为根标签

## 添加约束

添加约束有如下规则：

- 每个控制至少有两个约束：一个横向的一个纵向的
- 约束手柄只能与对应方向上的锚点链接，即横向的手柄跟横向的锚点链接，纵向的手柄跟纵向的锚点链接
- 每个约束手柄只能创建一个约束，但相同的锚点上可以被多个约束链接

向布局中添加控件很简单，只要通过拖拽就可以了。拖拽到布局中后，我们点击视图，会看到如下内容

![](http://7o4zgd.com1.z0.glb.clouddn.com/constraint02.png)

图中各部分分别是：

1 - 调整手柄：拖动手柄可调整视图大小
2 - 约束手柄：拖动可添加约束

鼠标放在圆圈上，如果未添加约束，圆圈会变为绿色，鼠标按住可进行拖拽，为此方向上添加约束；如果已添加约束，圆圈是红色的，点击可删除此方向上的约束。

相关属性：

- layout_constraintLeft_toLeftOf
- layout_constraintLeft_toRightOf
- layout_constraintRight_toLeftOf
- layout_constraintRight_toRightOf
- layout_constraintTop_toTopOf
- layout_constraintTop_toBottomOf
- layout_constraintBottom_toTopOf
- layout_constraintBottom_toBottomOf
- layout_constraintStart_toEndOf
- layout_constraintStart_toStartOf
- layout_constraintEnd_toStartOf
- layout_constraintEnd_toEndOf

3 - 清除按钮：点击可清除控件的所有约束
4 - 基线按钮（只有文字控件才有）：点击会在文字下方出现一个基线手柄，如下图

![constraint03](http://7o4zgd.com1.z0.glb.clouddn.com/constraint03.png)

基线只能与其他基线进行链接，实现与其他文字基线对齐的效果。

- layout_constraintBaseline_toBaselineOf

举例：

### 相对父控件位置

控制视图与父控件四周的位置关系

![与父控件的水平约束](http://7o4zgd.com1.z0.glb.clouddn.com/parent-constraint_2x.png)

### 按照顺序

使视图按照一定顺序摆放

![水平和垂直约束](http://7o4zgd.com1.z0.glb.clouddn.com/position-constraint_2x.png)

### 对齐

使一个视图的边缘与另一个视图的边缘对齐

![左对齐](http://7o4zgd.com1.z0.glb.clouddn.com/alignment-constraint_2x.png)
![左对齐带偏移](http://7o4zgd.com1.z0.glb.clouddn.com/alignment-constraint-offset_2x.png)

### 基准线对齐

使视图的文字基准线与另一个视图的文字基准线对齐

![基准线对齐](http://7o4zgd.com1.z0.glb.clouddn.com/baseline-constraint_2x.png)

### GuideLine (指引线/引导线) 约束

GuideLine 对用户来说是不可见的，有水平和竖直两种。我们可以按照距离布局边缘的百分比或实际距离来放置 GuideLine。

![guideline 约束](http://7o4zgd.com1.z0.glb.clouddn.com/guideline-constraint_2x.png)

点击如下按钮添加 Guideline 

![guideline 添加](http://7o4zgd.com1.z0.glb.clouddn.com/guideline.png)

已添加垂直方向 Guideline 为例，如下图，会出现一条垂直方向的虚线，虚线顶端有个圆圈，点击圆圈可以切换不同的模式：左边距、右边距、百分比，圆圈内图案会跟随发生变化，分别为向左箭头、向右箭头、百分号。

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/guidline_vertical.png)



## Properties 属性窗口

通过 Properties 窗口可以调整视图的一些属性，点击视图，它将会出现在编辑区域的右边，如下图所示：

![layout-editor-properties-callouts](http://7o4zgd.com1.z0.glb.clouddn.com/layout-editor-properties-callouts_2-3_2x.png)


各部分分别表示：

1 - 宽高比
2 - 删除约束
3 - 宽 / 高模式

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/layout-width-wrap.png) 代表Wrap Content

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/layout-width-match.png) 代表充满约束

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/layout-width-fixed.png) 代表固定尺寸

点击可在三个模式间切换

4 - 外边距
5 - 约束倾向

属性
- `layout_constraintHorizontal_bias`
- `layout_constraintVertical_bias`

值从0到1

当我们为视图两边（左右两边或上下两边）都添加约束时，视图会居中，默认为50%的倾向。

我们可以在编辑区域拖拽视图进行调整，或者在 `Properties`窗口进行调整。

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/bias.gif)


### 设置宽高比

为了设置宽高比，我们至少要将一个约束的尺寸设置为0dp，然后为 layout_constraintDimentionRatio 属性设置一个比例值，值可以是一个表示宽高比的浮点型数值，也可以是比例的形式：`width:height`例如

```
<Button
    android:layout_width="wrap_content"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="1:1"
    />
```

将使按钮的高度和宽度保持一致

You can also use ratio if both dimensions are set to MATCH_CONSTRAINT (0dp). In this case the system sets the largest dimensions the satisfies all constraints and maintains the aspect ratio specified. To constrain one specific side based on the dimensions of another. You can pre append W," or H, to constrain the width or height respectively. For example, If one dimension is constrained by two targets (e.g. width is 0dp and centered on parent) you can indicate which side should be constrained, by adding the letter W (for constraining the width) or H (for constraining the height) in front of the ratio, separated by a comma:

```
<Button 
    android:layout_width="0dp"
    android:layout_height="0dp"
    app:layout_constraintDimensionRatio="H,16:9"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintTop_toTopOf="parent"/>
```

will set the height of the button following a 16:9 ratio, while the width of the button will match the constraints to parent.


### 与 GONE 控件连接的外边距

两个控件 A 和 B，B 向 A 添加约束后，如果 A 的可见性变为 GONE，我可以通过设置相关属性，设置在 A 不可见时 B 的外边距，这些属性包括：

* layout_goneMarginStart
* layout_goneMarginEnd
* layout_goneMarginLeft
* layout_goneMarginTop
* layout_goneMarginRight
* layout_goneMarginBottom

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/relative-positioning-margin.png)

如图，B 向 A 添加约束后，如果 A 的可见性变为 GONE，B 的外边距就会变为 layout_goneMarginLeft 所指定的值。




## Chain Style

### 相关属性

* layout_constraintHorizontal_chainStyle
* layout_constraintVertical_chainStyle

### 包括如下类型

- spread - 所有元素展开
- spread_inside - 所有元素展开，但端点元素不展开，也就是两边的元素是紧邻边缘的
- packed - 元素作为一个整体，打包在一起, 可通过调整头元素的倾向，调整整体的倾向。


### 权重

属性：layout_constraintVertical_weight 

决定设置为 MATCH_CONSTRAINT （0dp）的元素，所占剩余空间的比例。

