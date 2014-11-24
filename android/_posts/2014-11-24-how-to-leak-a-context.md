---
layout: post
title: "Handlers和内部类是怎么使Context泄露的？"
modified: 2014-11-24 22:20:12
excerpt: "在使用Handler和内部类时，要注意一些问题，防止Context的泄露"
tags: [android, Tips]
comments: true
published : true
---

原文链接：[How to Leak a Context: Handlers & Inner Classes](http://www.androiddesignpatterns.com/2013/01/inner-class-handler-memory-leak.html)

首先看以下代码：

{% highlight java %}
public class SampleActivity extends Activity {

  private final Handler mLeakyHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
      // ... 
    }
  }
}
{% endhighlight%}

看起来不是很明显，这段代码会造成大量的内存泄露。Android Lint工具会出现如下警告：

>In Android, Handler classes should be static or leaks might occur.

但是泄露发生在哪？又是怎么发生的呢？在查明问题的根源之前，我们先看看我们所了解的东西：

1. 当应用程序启动的时候，android框架会未主线程创建一个`Looper`对象，它实现了一个简单的消息队列，一个接一个的传递`Message`对象，所有主要的应用框架事件（如Activit生命周期方法的调用，按钮的点击事件等）都被包含在一个被添加到此消息队列中Message对象当中，主线程的Looper对象存在于整个应用的生命周期中。
2. 当在主线程中实例化一个Handler的时候，它就与此Looper的消息队列产生了联系。推送到消息队列中的消息保存着对Handler的引用，因此能够调用Handler的handleMessage(Message) 方法。
3. 在java中，非静态内部及匿名类，会保留对外部类的引用，相反静态内部类则不会

那么泄露是在哪发生的呢？这是很细微的，请看下面的代码：

{% highlight java %}
public class SampleActivity extends Activity {
 
  private final Handler mLeakyHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
      // ...
    }
  }
 
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
 
    // Post a message and delay its execution for 10 minutes.
    mLeakyHandler.postDelayed(new Runnable() {
      @Override
      public void run() { }
    }, 60 * 10 * 1000);
 
    // Go back to the previous Activity.
    finish();
  }
}
{% endhighlight%}

当Activity结束时，延迟的消息在执行之前会一直保留在主线程中达10分钟，这个消息保留着一个对Activity中Handler的引用，而Handler又持有一个对外部类（SampleActivity）的引用，此引用会一直持续到Message执行，因此会阻止垃圾回收器对activity进行回收，也会造成应用资源的阻塞。第15行的匿名Runnable对象也会造成这样的问题。非静态的匿名内部类会持有对外部类的引用，因此造成context阻塞。

为了解决这个问题，我们可以在新的文件中创建Handler对象，或者将其声明为静态内部类。如果在Handler内部想调用activity中得方法，需要让Handler持有一个对activity的弱引用（WeakReference）。未解决匿名Runnable对象造成的内存泄露问题，我们也要声明未静态的。

{% highlight java %}
public class SampleActivity extends Activity {

  /**
   * Instances of static inner classes do not hold an implicit
   * reference to their outer class.
   */
  private static class MyHandler extends Handler {
    private final WeakReference<SampleActivity> mActivity;

    public MyHandler(SampleActivity activity) {
      mActivity = new WeakReference<SampleActivity>(activity);
    }

    @Override
    public void handleMessage(Message msg) {
      SampleActivity activity = mActivity.get();
      if (activity != null) {
        // ...
      }
    }
  }

  private final MyHandler mHandler = new MyHandler(this);

  /**
   * Instances of anonymous classes do not hold an implicit
   * reference to their outer class when they are "static".
   */
  private static final Runnable sRunnable = new Runnable() {
      @Override
      public void run() { }
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Post a message and delay its execution for 10 minutes.
    mHandler.postDelayed(sRunnable, 60 * 10 * 1000);
    
    // Go back to the previous Activity.
    finish();
  }
}
{% endhighlight%}

非静态内部类和静态内部类的差别是非常微妙的，它是每个android开发人员都应该理解的。要记住这条底线：避免在Activity中使用比其声明周期长的非静态内部类。
