---
layout: post
title: “优化布局结构"
modified: 2015-08-12 21:12:55
excerpt: ""
tags: [android, training, performance]
published: false
---

添加到布局中的每个控件和布局都需要初始化、布局、绘制。用嵌套的`LinearLayout`会造成过深的视图结构层级。而且使用`layout_weight`嵌套的LinearLayout代价是很高的，因为其中的每个控件都要被测算两次。在诸如`GridView`和`ListView`这种需要重复填充的视图中，这个问题也尤为重要。





