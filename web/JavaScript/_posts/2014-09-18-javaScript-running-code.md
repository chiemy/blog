---
layout: post
type: web
title: "[JavaScript]代码运行"
modified: 2014-9-18 22:46:27
excerpt: "引入JavaScript代码的方式"
tags: [web, js, javascript]
comments: true
---
有三种写`JavaScript`代码的方式。

##外部文件
第一种推荐的写代码的方式是在一个外部的文件（.js扩展名）中，可以在HTML中通过`<script>`标签引入，用`scr`属性指向文件的位置。把JavaScript保存到单独的文件中，可以提高代码的复用性。也可以使远程客户端上的浏览器缓存此文件，减少网页的加载时间。

##内联
另一种可选的方式是直接在网页上内联代码。也是通过`<script>`标签的方式，代码在标签之间，而不是通过`src`属性指定。尽管有使用这种方式的情形，但最好还是按照第一种方式。

{% highlight html %}
<!-- Embed code directly on a web page using script tags. -->
<script>
  alert( "Hello World!" );
</script>
{% endhighlight %}

##通过属性指定
最后一种方式是通过HTML元素的事件控制属性。这种方法强烈不建议使用。

{% highlight html %}
<!-- Inline code directly on HTML elements being clicked. -->
<a href="javascript:alert( 'Hello World' );">Click Me!</a>
<button onClick="alert( 'Good Bye World' );">Click Me Too!</button>
{% endhighlight %}

##部署
前两种方式的部署是很重要的，并且可以根据情况而异。如果你的`JavaScript`代码不访问当前网页的元素，你可以很安全的把它放置到HTML的`<head>`标签内。如果需要和网页元素进行交互，你需要保证这些元素在代码执行的时候存在。可以在下边的例子中看到这个陷阱，通过ID寻找元素的代码，将会在元素定义之前执行。

{% highlight html %}
<!doctype html>
<html>
<head>
    <script>
    // Attempting to access an element too early will have unexpected results.
    var title = document.getElementById( "hello-world" );
    console.log( title );
    </script>
</head>
<body>
 
<h1 id="hello-world">Hello World</h1>
 
</body>
</html>
{% endhighlight %}

将代码移到page的最低端是一种通常的做法，在`<body>`标签结束之前。这就保证了代码执行时，元素已经被定义了。

{% highlight html %}
<!doctype html>
<html>
<head></head>
<body>
 
<h1 id="hello-world">Hello World</h1>
<script>
// Moving the script to the bottom of the page will make sure the element exists.
var title = document.getElementById( "hello-world" );
console.log( title );
</script>
 
</body>
</html>
{% endhighlight %}