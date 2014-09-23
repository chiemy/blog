---
layout: post
type: web
title: "[JavaScript]初识"
modified: 2014-9-17 13:29:21
excerpt: "JavaScript简介"
tags: [web, js, javascript, 初识]
comments: true
---
原文地址：[JavaScript 101-Getting Started](http://learn.jquery.com/javascript-101/getting-started/)

##Web页剖析
在深入JavaScript之前，需要了解它是怎样和其他的网页技术协同工作的。

###HTML负责内容
HTML是用来定义和描述内容的标记语言。无论是博客上的文章、搜索引擎结果，还是电子商务网站，网页的核心内容都是用HTML实现的。

###CSS负责展现
CSS是为HTML文档提供样式支持的补充性语言。CSS通过定义字体，颜色和其他美学相关的内容使得内容看起来更好。CSS的强大之处在于，样式和内容的独立。这意味着你可以为同一项内容提供不同的样式，这也是响应式网站在不同的设备上看起来不错的决定性因素。

###JavaScript负责交互
在网页中，JavaScript为网页内容添加交互性和行为。如果没有JavaScript，网页将会是静态的，并且很无趣。 JavaScript为网页带来了生命。

下面是一个简单的HTML界面代码，包含了CSS和JavaScript，看看它们是如何结合到一起的。

<script src="https://gist.github.com/chiemy/2a8f10cc32886cf40c24.js?file=SamleHtml.html"></script>

在上例中，HTML是用来描述内容的。使用`<h1>`将文字“Hello World”标记为标题，“Click Me!”是对一个按钮的描述，按钮用`<button>`标签表示。`<style>`区块包含了能过修改标题字体大小和颜色的CSS。`<script>`区块包含了为按钮提供交互功能的JavaScript。当用户点击按钮的时候，会出现“Hello!”的提示信息。

##一种应用于Web的脚本语言
JavaScript最初是为网页添加交互性而设计的，不是一般意义上的编程语言，它是脚本语言。脚本语言比一般的语言更高效，因为他们为特定的领域（本例中就是web浏览器）做了优化。然而，JavaScript已经应用到了服务端（通过Node.js）,因此它可以替换如PHP, Ruby, ASP等一些语言。本指南只介绍通过jQuery，运行于浏览器中的JavaScript。

JavaScript的名称很容易让人误解，尽管和Java相似，它和java没有一点关系。JavaScript基于一个叫做“ECMAScript”的开放网络标准。基于标准的语言不被任何团体或公司所控制，是通过开发者共同定义的语言，这就是JavaScript能够在不同设备、不同浏览器上运行的原因。

##学习前的准备
- 浏览器
- 文本编辑器
- 开发工具（可选）

JavaScript最强大的优势之一就是<a title="n. 朴素；简易；天真；愚蠢">simplicity</a>，你可以再任何操作系统上编写或运行它，唯一的要求就是浏览器和文本编辑器。也有许多工具可以使得JavaScript的开发更加高效，但他们是可选的。

###开发工具
许多浏览器内置了开发工具，可以观察JavaScript和jQuery运行状态的功能。尽管它们不是必要的，但在调试错误时你会发现他们的用处。以下是一些开发工具：

- [Apple Safari](https://developer.apple.com/safari/tools/)
- [Google Chrome Developer Tools](https://developer.chrome.com/devtools)
- [Microsoft Internet Explorer](http://msdn.microsoft.com/en-us/library/ie/gg589507.aspx)
- [Mozilla Firefox Developer Tools](https://developer.mozilla.org/en-US/docs/Tools)
- [Opera Dragonfly](http://www.opera.com/dragonfly/)


