---
layout: post
title: "ConstraintLayout 的使用"
modified: 2017-03-19 19:35:17
excerpt: "使用 ConstraintLayout 创建更加偏平话的布局"
tags: [android, databinding]
published: true
---

<br>
ConstraintLayout （约束布局）在不嵌套布局的情况下，创建比较复杂的布局。它和 RelativeLayout 很相似，控件的位置都是跟兄弟控件和父控件相关联的，但它比 RelativeLayout 更加灵活，在 Android Studio 布局编辑中更容易使用。

在 Android Studio 布局编辑器的可视化工具下，ConstraintLayout 所有功能都可以使用。可以通过拖拽的方式来创建 ConstraintLayout 布局。

## 约束布局的添加

按照如下步骤添加：

1. 点击 **Tools > Android > SDK Manager**
2. 点击 **SDK Tools** 标签
3. 展开 **Support Repository** 项，然后选中 **ConstraintLayout for Android** 和 **Solver for ConstraintLayout**。选中 **Show Package Details** 记住版本，接下来要用到。
4. 点击 **OK**
5. 然后在应用的 `build.gradle` 下添加依赖

```
dependencies {
    compile 'com.android.support.constraint:constraint-layout:1.0.0-alpha7'
}
```
版本号为步骤3中记录下来的，现在还没有发布正式版。

6. 最后点击 **Sync Project with Gradle Files.**

### 将已存在的布局转换为约束布局

通过以下方式，可以将一个已经存在的布局转换为约束布局：

![constraintlayout01.png](http://7o4zgd.com1.z0.glb.clouddn.com/constraintlayout01.png)

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

![手柄](http://7o4zgd.com1.z0.glb.clouddn.com/constraint02.png)

图中各部分分别是：

1. 调整手柄：拖动手柄可调整视图大小
2. 约束手柄：拖动可添加约束

鼠标放在圆圈上，如果未添加约束，圆圈会变为绿色，鼠标按住可进行拖拽，为此方向上添加约束；如果已添加约束，圆圈是红色的，点击可删除此方向上的约束。

3. 清除按钮：点击可清除控件的所有约束
4. 基线按钮（只有文字控件才有）：点击会在文字下方出现一个基线手柄，如下图

![手柄](http://7o4zgd.com1.z0.glb.clouddn.com/constraint03.png)

基线只能与其他基线进行链接，实现与其他文字基线对齐的效果。

我们可以使用约束达到以下布局效果：

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

### GuideLine (指引线) 约束

GuideLine 对用户来说是不可见的，有水平和竖直两种。我们可以按照距离布局边缘的百分比或实际距离来放置 GuideLine。

![guideline 约束](http://7o4zgd.com1.z0.glb.clouddn.com/guideline-constraint_2x.png)

点击如下按钮添加 Guideline 

![guideline 添加](http://7o4zgd.com1.z0.glb.clouddn.com/guideline.png)

已添加垂直方向 Guideline 为例，如下图，会出现一条垂直方向的虚线，虚线顶端有个圆圈，点击圆圈可以切换不同的模式：左边距、右边距、百分比，圆圈内图案会跟随发生变化，分别为向左箭头、向右箭头、百分号。

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/guidline_vertical.png)



## 调整约束倾向

当我们为视图两边（左右两边或上下两边）都添加约束时，视图会居中，默认为50%的倾向。

我们可以在编辑区域拖拽视图进行调整，或者在 `Properties`窗口进行调整。

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/bias.gif)


## 调整视图大小

![guideline vertical](http://7o4zgd.com1.z0.glb.clouddn.com/layout-editor-properties-callouts_2-3_2x.png)









