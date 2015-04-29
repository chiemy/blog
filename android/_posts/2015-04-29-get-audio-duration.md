---
layout: post
title: "获取mp3文件的播放时长"
modified: 2015-04-29 22:03:09
excerpt: "在未开始播放mp3文件之前，应该如何获取播放时长"
tags: [android, media]
comments: true
---
最近在修改一个关于音频播放的项目，涉及到音频信息的展示，包括音频名称，时长。于是我看到了像下面这样的代码：

{% highlight java %}

	/**
	 * 获 取文件时间长度
	 * 
	 * @return
	 */
	 
	 public static int getRecordLong(String filePath) {
		MediaPlayer mp = null;
		try {
			mp = new MediaPlayer();
			mp.setDataSource(firePath);
			mp.prepare();
			mp.start();
		} catch (IllegalArgumentException e) {
			e.printStackTrace();
		} catch (SecurityException e) {
			e.printStackTrace();
		} catch (IllegalStateException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		
		int duration = mp.getDuration();
		if(mp != null) {
			mp.stop();
			mp.release();
			mp = null;
		}
		return duration;
	}
{% endhighlight%}

看到这样的代码我也是醉了，我说写这个代码的哥们，为了获取一个播放时长有必要如此大动干戈？而且这段代码竟还是在主线程中调用的，不由得倒吸了一口凉气，必须马上改造。

##如何获取文件时长？
像上述方式，只适用获取播放中的文件时长，如果只是为了展示一下时间而采用上述方式，显然太消耗性能，太大才小用了。那在我们知道文件路径的情况下，如何获取播放时长呢？

###方式一：使用ContentResolver
系统的ContentResolver为我们提供了查询媒体库的方法，在媒体库里保存着一些媒体文件的信息，如名称、文件路径、播放时长等信息。我们可以通过它来获取相应媒体文件的播放时长。当然首先要保证媒体库里有该文件的信息（关于媒体库更新，见[Android媒体库更新问题研究](http://chiemy.com/android/meida-refresh/)）

{% highlight java %}

	private Uri contentUri = Media.EXTERNAL_CONTENT_URI;
	private String[] columns = { MediaStore.Audio.Media._ID, // 歌曲ID
			MediaStore.Audio.Media.DURATION,// 歌曲的总播放时长
	};
	
	private String getDurationContext context, String path) {
		String s = "00:00";
		ContentResolver mResolver = context.getContentResolver();
		String selection = MediaStore.Audio.Media.DATA + "=?";
		String selectionArgs[] = {path};
		Cursor cursor = mResolver.query(contentUri, null, selection, selectionArgs,
				null);
		if(cursor != null && cursor.moveToFirst()){
			int duration = cursor.getInt(cursor.getColumnIndex(Media.DURATION));
			s =  StringUtils.formatDuration(duration);
			cursor.close();
		}
		return s;
	}
{% endhighlight%}

###方式二：使用MediaMetadataRetriever类
`MediaMetadataRetriever`类提供了从媒体文件获取帧(frame)和元数据（meta data）的统一接口。如获取视频在某一时间点得画面`getFrameAtTime(long timeUs)`，此方法会返回一个`Bitmap`对象。

也可以通过它获取媒体文件时长，具体操作如下(Api 10+)：

{% highlight java %}

	@SuppressLint("NewApi") 
	private long getDuration(String path) {
		MediaMetadataRetriever retriever = new MediaMetadataRetriever();
		retriever.setDataSource(path); //在获取前，设置文件路径（应该只能是本地路径）
		String duration =  
				retriever.extractMetadata(MediaMetadataRetriever.METADATA_KEY_DURATION);
		retriever.release(); //释放
		if(TextUtils.isEmpty(duration)){
			return "00:00";
		}
		long dur = Long.parseLong(duration);
		return dur;
	}
{% endhighlight%}

