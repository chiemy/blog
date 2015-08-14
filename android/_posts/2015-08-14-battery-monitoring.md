---
layout: post
title: "监控电量和充电状态"
modified: 2015-08-13 21:12:55
excerpt: ""
tags: [android, training, performance]
published: true
---

### 查询当前充电状态

 `BatteryManager` 会广播一个包含所有电池和充电信息的粘性的intent（sticky intent），其中就包含当前充电状态。

由于是粘性的intent，我们不需要注册一个广播接收器，在调用`registerReceiver`方法时传入`null`作为接收器即可，利用方法返回的`intent`即可获取电池状态，下面是示例代码：

{%highlight java%}

IntentFilter ifilter = new IntentFilter(Intent.ACTION_BATTERY_CHANGED);

Intent batteryStatus = context.registerReceiver(null, ifilter);

{%endhighlight%}

然后我们可以获取当前的充电状态，还可以知道是通过USB还是通过充电器（AC）进行充电。

{%highlight java%}

// 在充电还是已充满？

int status = batteryStatus.getIntExtra(BatteryManager.EXTRA_STATUS, -1);

boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||

                     status == BatteryManager.BATTERY_STATUS_FULL;

// 通过什么充电的？

int chargePlug = batteryStatus.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);

boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;

boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;

{%endhighlight%}

### 监控充电状态的改变

在设备与电源连接或断开时，`BatteryManager`会对相应的事件进行广播。如果应用需要对充电状态的指示，应当在manifest文件中注册一个广播，来监听这两个事件。

{%highlight xml%}

<receiver android:name=".PowerConnectionReceiver">

  <intent-filter>

    <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>

    <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>

  </intent-filter>

</receiver>

{%endhighlight%}

在广播接收器中，我们可以监听到这两个事件

{%highlight xml%}

  public class PowerConnectionReceiver extends BroadcastReceiver {

    @Override

    public void onReceive(Context context, Intent intent) { 

        int status = intent.getIntExtra(BatteryManager.EXTRA_STATUS, -1);

        boolean isCharging = status == BatteryManager.BATTERY_STATUS_CHARGING ||

                            status == BatteryManager.BATTERY_STATUS_FULL;

    

        int chargePlug = intent.getIntExtra(BatteryManager.EXTRA_PLUGGED, -1);

        boolean usbCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_USB;

        boolean acCharge = chargePlug == BatteryManager.BATTERY_PLUGGED_AC;

    }

}

{%endhighlight%}

### 获取当前电量

和查询当前充电状态的方法类似：

{%highlight java%}

int level = batteryStatus.getIntExtra(BatteryManager.EXTRA_LEVEL, -1);

int scale = batteryStatus.getIntExtra(BatteryManager.EXTRA_SCALE, -1);

float batteryPct = level / (float)scale;

{%endhighlight%}

### 监控重要的电量变化事件

持续监控电池电量，要比应用正常的行为对电池的影响大，因此不建议持续的监控。我们只需要监控一些重要事件就可以了，尤其是设备进入或退出低电量状态的事件。

下面是接收进入或退出低电量状态的事件的代码示例：

{%highlight xml%}

<receiver android:name=".BatteryLevelReceiver">

<intent-filter>

  <action android:name="android.intent.action.ACTION_BATTERY_LOW"/>

  <action android:name="android.intent.action.ACTION_BATTERY_OKAY"/>

  </intent-filter>

</receiver>

{%endhighlight%}











