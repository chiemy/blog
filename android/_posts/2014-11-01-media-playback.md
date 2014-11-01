---
layout: post
title: "媒体播放"
modified: 2014-11-01 19:35:05
excerpt: "在最近的项目中涉及到了音频播放，参照Android API Guides，对MediaPlayer的使用做一个简要的说明。"
tags: [android, Media]
comments: true
published : true
---
<section id="table-of-contents" class="toc">
  <header>
    <h3>主要内容</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

在最近的项目中涉及到了音频播放，参照Android API Guides，对MediaPlayer的使用，做一个简要的说明。

##1.权限声明

在开始使用`MediaPlayer`之前，我们可能需要在Manifest中声明一些权限。

1.如果涉及到网络播放，那么需要加入网络权限：

{% highlight xml %}
	<uses-permission android:name="android.permission.INTERNET" />
{% endhighlight %}
	
这个权限我们已经非常熟悉了。

2.如果你的应用想保持屏幕常亮或者不让设备休眠，或需要使用`MediaPlayer.setScreenOnWhilePlaying()`或者`MediaPlayer.setWakeMode()`方法，必须加入[Wake Lock](http://developer.android.com/reference/android/Manifest.permission.html#WAKE_LOCK)权限：

{% highlight xml %}
	<uses-permission android:name="android.permission.WAKE_LOCK" />
{% endhighlight %}
	
##2.MediaPlayer的使用
###2.2.不同资源种类的基本用法
`MediaPlayer`类是媒体框架中最重要的组件之一。播放资源分为以下几种：

- 本地资源
- 内部URIs，例如从Content Resolver获取的URI.
- 外部URI(流)，一般是网络。

>点击查看：[Android支持的媒体格式](http://developer.android.com/guide/appendix/media-formats.html#recommendations)

####2.2.1.播放保存在`res/raw/`目录下的媒体资源
使用示例：

{% highlight java %}
	MediaPlayer mediaPlayer = MediaPlayer.create(context, R.raw.sound_file_1);
	mediaPlayer.start(); //不需要调用 prepare(); create()方法
{% endhighlight %}
	
与以下两种方式区别较大。
	
####2.2.2.播放设备内的资源
保存在媒体库中的资源，可以通过如下方式

{% highlight java %}
	Uri myUri = ....; //通过Content Resovler获取
	MediaPlayer mediaPlayer = new MediaPlayer();
	mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
	mediaPlayer.setDataSource(getApplicationContext(), myUri);
	mediaPlayer.prepare();
	mediaPlayer.start(); //prepare()回调后才能调用
{% endhighlight %}
	
>注：对`setAudioStreamType`作用不是很了解，文档中叶没有说明，只说了必须在`prepare()`之前调用，否则无效。
初步感觉，它与音量有些关系，因为在系统设置中我们可以对闹钟、音乐、铃声设置不同的音量，通过`setAudioStreamType`方法设置了不同类型，可能音量就会不同。当然可能还会有更深层次的作用，暂时还没有找到答案。
	
如果知道资源的明确路径：

{% highlight java %}
	String path = ....;
	....
	mediaPlayer.setDataSource(path);
	....
{% endhighlight %}
	
####2.2.3通过HTTP流播放远程URL

{% highlight java %}
	String url = "http://........";
	MediaPlayer mediaPlayer = new MediaPlayer();
	mediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
	mediaPlayer.setDataSource(url);
	mediaPlayer.prepare(); // 耗时较长! (因为缓冲等操作)
	mediaPlayer.start(); 
{% endhighlight %}

###2.3.注意点
###2.3.1.Prepare的异步调用
因为prepare过程会获取和解码数据，所以是一个很耗时的操作。`prepare()`并不是异步的，因此绝不要在UI线程中调用此方法，即使你感觉这个过程是很快的。有两个选择：自己在子线程中处理，或者使用它的异步版本`prepareAsync()`，这个方法会异步的进行后台操作，不用担心阻塞主线程。

###2.3.2.状态管理
一部电梯，有开门、关门、运行等状态，开门状态下是绝对不可以运行的，或则可能死人啊。和电梯类似，MediaPlayer也是有内部状态的，从哪个状态能过度到哪个状态都是有要求的，如果使用不当，虽然不会造成死人的后果，但可是会死程序的啊，切记，切记！

当你创建完一个MediaPlayer，它处于`Idle`（空闲）状态，通过setDataResource()方法之后，处于`Initialized`(初始化)状态，之后你需要调用prepare()或prepareAsync()方法，当MediaPlayer完成准备工作之后，会进入到`Prepared`状态，这时你就可以调用start()方法了。然后，我们就可以通过start(),pause()和seekTo()等方法，在`Started`,`Paused`,`PlaybackCompleted`状态间来回切换了。当你调用stop()方法后，直到下次到达`Prepared`状态之前，是不能调用start()方法的。

当然实际的情况，要比这复杂的多，下面是状态间过度的流程图：

![image](http://chiemyblog.qiniudn.com/mediaplayer_state_diagram.gif)

图中有两种类型的线。由一个箭头开始的线代表同步的方法调用，而以双箭头开头的代表的线代表异步方法调用。

###2.3.3.错误处理

在一般情况下，由于种种原因一些播放控制操作可能会失败，如不支持的音频/视频格式，流超时等原因，等等。因此，错误报告和恢复在这种情况下是非常重要的。有时，由于编程错误，在处于无效状态的情况下调用了一个播放控制操作可能发生。在所有这些错误条件下，内部的播放引擎会调用一个由客户端程序员提供的OnErrorListener.onError()方法。客户端程序员可以通过调用MediaPlayer.setOnErrorListener（android.media.MediaPlayer.OnErrorListener）方法来注册OnErrorListener.

> 注意：
> 
> - 一旦发生错误，MediaPlayer对象会进入到Error状态。
> - 为了重用一个处于Error状态的MediaPlayer对象，可以调用reset()方法来把这个对象恢复成Idle状态。
> - 注册一个OnErrorListener来获知内部播放引擎发生的错误是好的编程习惯。
> - 在不合法的状态下调用一些方法，如prepare()，prepareAsync()和setDataSource()方法会抛出IllegalStateException异常。

	
###2.3.4.MediaPlayer的释放
MediaPlayer会消耗很多系统资源，因此不要在你不需要的时候长时间保持一个MediaPlayer实例。当你使用完成之后一定要调用`release()`方法，释放资源。

{% highlight java %}
	mediaPlayer.release();
	mediaPlayer = null;
{% endhighlight %}
	
	
##3.和Service一同使用
如果想让我们的应用保持后台播放的能力，我们就需要在Service中对MediaPlayer进行控制了。

代码示例：

{% highlight java %}
	public class MyService extends Service implements MediaPlayer.OnPreparedListener {
    	private static final String ACTION_PLAY = "com.example.action.PLAY";
    	MediaPlayer mMediaPlayer = null;

    	public int onStartCommand(Intent intent, int flags, int startId) {
        	...
        	if (intent.getAction().equals(ACTION_PLAY)) {
            	mMediaPlayer = ... // initialize it here
            	mMediaPlayer.setOnPreparedListener(this);
            	mMediaPlayer.prepareAsync(); // prepare async to not block main thread
        	}
    	}

    	/** Called when MediaPlayer is ready */
    	public void onPrepared(MediaPlayer player) {
        	player.start();
    	}
	}
{% endhighlight %}

###3.1.作为前台服务（foreground service）使用
后台服务一般适用于一些不需要用户知悉的操作，如同步数据，下载内容等。但似乎不太适合音乐播放，因为即使是在后台播放音乐，用户还是想保持与应用的交互能力，当用户想停止音乐时，只能通过点击桌面应用图标，找到音乐播放界面，并按下那个暂停按钮才行吗？遇到这样的应用，我肯定会卸载掉的！

让服务作为前台服务，我们需要借助Notifycation，在Service中调用`startForeground()`方法，代码示例：

{% highlight java %}
	String songName;
	// assign the song name to songName
	PendingIntent pi = PendingIntent.getActivity(getApplicationContext(), 0,
                new Intent(getApplicationContext(), MainActivity.class),
                PendingIntent.FLAG_UPDATE_CURRENT);
	Notification notification = new Notification();
	notification.tickerText = text;
	notification.icon = R.drawable.play0;
	notification.flags |= Notification.FLAG_ONGOING_EVENT;
	notification.setLatestEventInfo(getApplicationContext(), "MusicPlayerSample",
                "Playing: " + songName, pi);
	startForeground(NOTIFICATION_ID, notification);
{% endhighlight %}

如此你的定义的Notifycation就能出现在通知栏了，在不需要的时候调用`stopForeground(true)`，使其不在通知栏显示。

##4.处理声音焦点
因为Andorid是支持多任务的，可能会有很多程序需要播发音效，所以问题来了，如果多个程序播放音频，那岂不是乱套了？在Android 2.2之前，确实没有什么内建的机制解决这个问题。在Android 2.2之后，平台提供了一种方式，来协调各程序对音频输出端的使用，叫做`Audio Focus`。

当你的应用需要输出音效时，你的应用需要请求获取焦点，一旦获取了焦点，你就可以自由的输出了。但同时，我们也要对焦点的变化进行监听，当我们失去焦点时，需要结束我们的音频输出或者调低音量（取决于失去焦点的时常，如果是短时的失去焦点，我们可以适当调低音量，如果是长时的，那么我们需要结束音频输出）。

请求焦点：

{% highlight java %}
	AudioManager audioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
	int result = audioManager.requestAudioFocus(this, AudioManager.STREAM_MUSIC,
    	AudioManager.AUDIOFOCUS_GAIN);

	if (result != AudioManager.AUDIOFOCUS_REQUEST_GRANTED) {
		// could not get audio focus.
	}
{% endhighlight %}

`requestAudioFocus()`方法的第一个参数是`AudioManager.OnAudioFocusChangeListener`，它的`onAudioFocusChange(int focusChange)`会在焦点发生变化时被调用。`focusChange`的取值和含义如下：

- AUDIOFOCUS_GAIN: 获取到了焦点
- AUDIOFOCUS_LOSS: 失去了焦点，长时间后才能获得，此时我们应该释放播放相关资源。
- AUDIOFOCUS_LOSS_TRANSIENT: 暂时失去焦点，稍后还会重新获取，此时我们可以选择暂停播放，不用释放资源。
- AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK: 暂时失去焦点，但我们可以选择将声音调小，一般来短信会触发。

代码示例：

{% highlight java %}
	public void onAudioFocusChange(int focusChange) {
    	switch (focusChange) {
        	case AudioManager.AUDIOFOCUS_GAIN:
            	// resume playback
            	if (mMediaPlayer == null) initMediaPlayer();
            	else if (!mMediaPlayer.isPlaying()) mMediaPlayer.start();
            		mMediaPlayer.setVolume(1.0f, 1.0f);
            	break;

        	case AudioManager.AUDIOFOCUS_LOSS:
            	// Lost focus for an unbounded amount of time: stop playback and release media player
            	if (mMediaPlayer.isPlaying()) mMediaPlayer.stop();
            	mMediaPlayer.release();
            	mMediaPlayer = null;
            break;

        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT:
            	// Lost focus for a short time, but we have to stop
           		// playback. We don't release the media player because playback
            	// is likely to resume
            	if (mMediaPlayer.isPlaying()) mMediaPlayer.pause();
            break;

        case AudioManager.AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
            	// Lost focus for a short time, but it's ok to keep playing
            	// at an attenuated level
            	if (mMediaPlayer.isPlaying()) mMediaPlayer.setVolume(0.1f, 0.1f);
            break;
    	}
    }
{% endhighlight %}
    
工具类：

{% highlight java%}
public class AudioFocusHelper implements AudioManager.OnAudioFocusChangeListener {
    AudioManager mAudioManager;

    // other fields here, you'll probably hold a reference to an interface
    // that you can use to communicate the focus changes to your Service

    public AudioFocusHelper(Context ctx, /* other arguments here */) {
        mAudioManager = (AudioManager) mContext.getSystemService(Context.AUDIO_SERVICE);
        // ...
    }

    public boolean requestFocus() {
        return AudioManager.AUDIOFOCUS_REQUEST_GRANTED ==
            mAudioManager.requestAudioFocus(mContext, AudioManager.STREAM_MUSIC,
            AudioManager.AUDIOFOCUS_GAIN);
    }

    public boolean abandonFocus() {
        return AudioManager.AUDIOFOCUS_REQUEST_GRANTED ==
            mAudioManager.abandonAudioFocus(this);
    }

    @Override
    public void onAudioFocusChange(int focusChange) {
        // let your service know about the focus change
    }
}
{% endhighlight %}


###5.处理AUDIO_BECOMING_NOISY意图
在某些情况下，我们的音乐会显得很吵闹，例如，你正使用耳机在公司里听音乐，此时，你无意中将耳机拔掉了，突然冒出来的声音把你的同事吓了一跳。像这种情况显然是我们不想发生的。我们只需要注册一个广播接收器，来接收相应的事件进行处理就可以了。

在Manifest中注册：

{% highlight xml %}
<receiver android:name=".MusicIntentReceiver">
   <intent-filter>
      <action android:name="android.media.AUDIO_BECOMING_NOISY" />
   </intent-filter>
</receiver>
{% endhighlight %}

代码中进行处理：

{% highlight java %}
public class MusicIntentReceiver extends android.content.BroadcastReceiver {
   @Override
   public void onReceive(Context ctx, Intent intent) {
      if (intent.getAction().equals(
                    android.media.AudioManager.ACTION_AUDIO_BECOMING_NOISY)) {
          //可以停止播放啦
      }
   }
}
{% endhighlight %}

###6.从Content Resolver中检索音频

{% highlight java %}
ContentResolver contentResolver = getContentResolver();
Uri uri = android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI;
Cursor cursor = contentResolver.query(uri, null, null, null, null);
if (cursor == null) {
    // query failed, handle error.
} else if (!cursor.moveToFirst()) {
    // no media on the device
} else {
    int titleColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media.TITLE);
    int idColumn = cursor.getColumnIndex(android.provider.MediaStore.Audio.Media._ID);
    do {
       long thisId = cursor.getLong(idColumn);
       String thisTitle = cursor.getString(titleColumn);
       // ...process entry...
    } while (cursor.moveToNext());
}
{% endhighlight %}

可以使用MediaPlayer播放检索到的音频啦：

{% highlight java %}
long id = /* retrieve it from somewhere */;
Uri contentUri = ContentUris.withAppendedId(
        android.provider.MediaStore.Audio.Media.EXTERNAL_CONTENT_URI, id);

mMediaPlayer = new MediaPlayer();
mMediaPlayer.setAudioStreamType(AudioManager.STREAM_MUSIC);
mMediaPlayer.setDataSource(getApplicationContext(), contentUri);

// ...prepare and start...
{% endhighlight %}

##未完待续……
内容预告-线控、蓝牙控制