---
layout: post
type: android
title: "[API Guide]蓝牙"
modified: 2014-9-29 15:29:51
excerpt: ""
tags: [android, bluetooth ,Android Api,翻译]
comments: true
published : false
---

Android平台提供了蓝牙网络模块的支持，允许设备间进行无线数据交换。通过Android蓝牙APIs,应用框架可以访问蓝牙。这些APIs允许应用程序和其他设备进行无线连接，包括点对点和多点通讯功能。

使用此APIs,Andriod应用可以进行如下操作：

- 扫描其他蓝牙设备
- 从本地蓝牙适配器中找到已配对设备
- 建立RFCOMM（串行电缆仿真协议）通道
- 通过Service discovery连接其他设备
- 和其他设备进行数据交互
- 管理多个连接

本文介绍传统蓝牙的使用方法，传统蓝牙更适合大电量的设备操作，例如流媒体，以及Android设备间的通讯。Android4.3引入了对低功耗蓝牙支持的API,想了解更多请查看[Bluetooth Low Energy.](http://developer.android.com/guide/topics/connectivity/bluetooth-le.html)

## 基础
本文档描述了怎样使用蓝牙APIs完成蓝牙通讯的四个主要任务：蓝牙设置，查找已配对或周围可用的蓝牙设备，连接设备，设备间传输数据。

所有的这些APIs都可以在`android.bluetooth`保中找到，下面是对你可能在创建蓝牙连接时用到的类和接口的简要描述：

`BluetoothAdapter`

代表着本地的蓝牙适配器（蓝牙无线电radio）,它是所有蓝牙交互的入口。使用它，你可以发现其他的蓝牙设备，查询已配对的设备列表，使用已知的MAC地址实例化`BluetoothDivice`,并创建一个`BluetoothServerSocket`来监听其他设备的连接。

`BluetoothDevice`

代表一个远程的蓝牙设备，使用它来请求与远程设备通过`BluetoothSocket`的连接，或者查询例如设备名称、地址、类以及绑定状态等设备信息。

`BluetoothSocket`

代表蓝牙socket接口（类似Tcp socket）。它是允许应用程序与其他蓝牙设备通过输入、输出流交换数据的连接点。

`BluetoothServerSocket`
代表着开放的server socket，监听传入请求（和Tcp ServerSocket类似）。为了两个设备能够实现连接，其中一个必须使用此类开启一个开放的server socket。当一个远程的蓝牙设备向这个设备发起连接请求时，当此设备接受连接请求后，`BluetoothServerSocket`会返回连接好的`BluetoothSocket`。

`BluetoothClass`
用来描述蓝牙设备的特征和能力。它是个只读的属性集合，定义了设备的主要及副设备类及其服务。然而，对设备所有的蓝牙配置和服务描述并不一定可靠，但是对设备类型的提示是有用的。

`BluetoothProfile`

一个代码蓝牙模式(profile)的接口。蓝牙模式是设备间蓝牙交互的无线接口协议。其中一个例子是Hands-Free（免提）模式。更多关于`profiles`的讨论见[Profiles的使用](#Profiles)

`BluetoothHeadset`

提供对蓝牙耳机的支持。它既包括蓝牙耳机也包括Hands-Free(V1.5)模式。


`BluetoothA2dp`

定义了通过蓝牙连接的设备间传输流的品质。`A2DP`代表` Advanced Audio Distribution Profile`(高级音频分发模式)。

`BluetoothHealth`

代表一个控制蓝牙设备的一个健康设备模式代理（Health Device Profile）。

`BluetoothHealthCallback`

一个可以用来实现`BluetoothHealth`的回调抽象类。继承这个类实现其回调方法，用来接收应用注册状态和蓝牙信道状态的改变。

`BluetoothHealthAppConfiguration`

Represents an application configuration that the Bluetooth Health third-party application registers to communicate with a remote Bluetooth health device.

`BluetoothProfile.ServiceListener`

通知`BluetoothProfile`进程间通讯客户端，已经与服务（运行于特殊配置文件中的内部服务）连接或断开连接的接口。


##蓝牙权限
为了在应用中使用蓝牙，你必须声明蓝牙权限`BLUETOOTH`。任何蓝牙通讯的操作都需要此权限，例如，请求连接，接收连接，以及传输数据。

如果你想让你的应用启动设备发现或者修改蓝牙配置，那么必须同时声明`BLUETOOTH_ADMIN`权限。许多应用程序需要这个权限仅仅用来来发现本地的蓝牙设备。其他功能并不能被使用，除非是一个电源管理的应用，在取得用户许可后可以修改设置。

><strong>注意：</strong>如果你声明了`BLUETOOTH_ADMIN`权限，那么同时也要申请`BLUETOOTH`权限。

{% highlight xml %}
<manifest ... >
  <uses-permission android:name="android.permission.BLUETOOTH" />
  ...
</manifest>
{% endhighlight %}

##设置蓝牙
在应用可以通过蓝牙进行通讯之前，你需要保证在设备上支持蓝牙，如果支持，还要保证是可用的。

如果设备不支持蓝牙，那么你就需要禁用蓝牙功能。如果支持蓝牙，但是不可用，那么在不离开自己应用的前提下，我们可以要求用户开启蓝牙。此设置需要使用`BluetoothAdapter`两步完成。

1.获取`BluetoothAdapter`
`BluetoothAdapter`是任何蓝牙活动都需要的,通过调用`getDefaultAdapter()`来获取`BluetoothAdapter`对象，此方法返回一个代表设备自身的蓝牙适配器（蓝牙无线电）。整个系统只有一个蓝牙适配器，可以使用这个类与之进行交互。如果`getDefaultAdapter()`返回了null，那么说明此设备不支持蓝牙。

{% highlight java %}
BluetoothAdapter mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
if (mBluetoothAdapter == null) {
    // Device does not support Bluetooth
}
{% endhighlight %}

2.开启蓝牙
下一步，你需要保证蓝牙是开启的。调用`isEnable()`方法来检查蓝牙是否开启。如果返回false,那么蓝牙没有开启。我们需要调用`startActivityForResult()`方法，并传递一个action为`ACTION_REQUEST_ENABLE`的意图，来请求开启蓝牙。这将发送一个通过系统设置来开启蓝牙的请求。例如：

{% highlight java %}
if (!mBluetoothAdapter.isEnabled()) {
    Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
}
{% endhighlight%}

此时会出现一个请求用户开启蓝牙的对话框，如果用户点击“确定”，那么系统将会开启蓝牙，焦点会重新回到你的应用。

`REQUEST_ENABLE_BT`是个本地定义的整数常量（必须大于0），系统将会将其作为`requestCode`参数返回给`onActivityResult()`方法。

如果开启蓝牙成功，会在`onActivityResult()`中收到为`RESULT_OK`的结果码，否则为`RESULT_CANCELED`

我们也可以监听意图action为`ACTION_STATE_CHANGED`的广播，来获取蓝牙的状态。广播包含额外的字段`EXTRA_STATE`和`EXTRA_PREVIOUS_STATE`，分别包含新的和旧的蓝牙状态。这些额外属性的可能值有`STATE_TURNING_ON`, `STATE_ON`, `STATE_TURNING_OFF`。

>小贴士：开启可被发现功能，会自动的开启蓝牙，如果你在开始蓝牙活动执行想一直保持蓝牙可被发现，那么可以跳过上边的两步。请阅读[可被发现](#enable_discoverability)。

##查找设备
使用`BluetoothAdapter`可以通过设备发现功能或者查询配对设备列表的方式，获取远程蓝牙设备。

设备搜寻是一个查询周边蓝牙开启的设备并请求一些设备信息的扫描过程。然而，被扫描的设备必须处于可被发现的状态下，才能对查询操作做出回应。如果一个设备是可被发现的，它会反馈给发现请求一些信息，比如设备名称、类以及唯一的MAC地址。使用这些信息，发起发现请求的设备就可以与被发现的设备建立连接了。

一旦与远程设备第一次建立连接，会向用户展现一个配对请求。当设备配对后，关于设备的基本信息（比如设备名称、类以及唯一的MAC地址）就会被保存，并且可以通过Bluetooth APIs进行读取。使用远程设备的MAC地址，在不用扫描发现的情况下可以随时建立连接（当然要在范围内）。

配对和连接之间是不同的，配对是设备之间知晓对方的存在，他们有一个用户认证的共享的`link-key`，之间可以建立一个加密连接。连接意味着，设备分享一个`RFCOMM`信道，用来进行传输数据。当前的Android Bluetooth APIs要求在`RFCOMM`信道建立之前需要进行配对。（当使用Bluetooth APIs建立加密连接时，配对是自动进行的）。

下面的部分介绍，如何发现已配对设备，或使用设备发现功能发现新的设备。

>注意:Android驱动的设备，默认情况下是不可被发现的。用户可以通过系统设置，让设备在一定时间内可被发现，或者可以在不离开当前应用的前提下，可以请求用户开启可被发现功能。具体见下文中的[开启可被发现功能](#enable_discoverability)

###查询配对设备
在进行设备搜寻操作之前，我们最好查询一下我们期望的设备是否已经在配对列表中。通过`getBondedDevices()`既可获取。方法会返回代表以配对的`BluetoothDevice`的集合。例如，你可以查询所有的已配对设备，然后将设备名称展示给用户：

{% highlight java %}
Set<BluetoothDevice> pairedDevices = mBluetoothAdapter.getBondedDevices();
// 是否存在已配对设备
if (pairedDevices.size() > 0) {
 	//循环读取配对设备
 	for (BluetoothDevice device : pairedDevices) {
 		//展示设备名称和MAC地址
 		mArrayAdapter.add(device.getName() + "\n" + device.getAddress());
 	}
}
{% endhighlight %}

###搜寻设备
通过调用`startDiscovery()`方法，开始搜寻设备。此过程是异步的，并且会马上返回一个表示搜寻是否成功开始的布尔值。这个过程通常会进行12秒左右，之后是每个被发现的设备检索蓝牙名称的page scan。

我们需要为`ACTION_FOUND`的意图注册一个广播接收器，来获取每个被发现设备的信息。没发现一个，系统就会发送一个包含`ACTION_FOUND`的意图的广播。此意图携带了`EXTRA_DEVICE`和`EXTRA_CLASS`属性，分别包含`BluetoothDevice`和`BluetoothClass`。举例：

{% highlight java %}
// Create a BroadcastReceiver for ACTION_FOUND
private final BroadcastReceiver mReceiver = new BroadcastReceiver() {
 	public void onReceive(Context context, Intent intent) {
 		String action = intent.getAction();
 		// 当发现一个设备时
 		if (BluetoothDevice.ACTION_FOUND.equals(action)) {
 			// Get the BluetoothDevice object from the Intent
 			BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
 			// Add the name and address to an array adapter to show in a ListView
 			mArrayAdapter.add(device.getName() + "\n" + device.getAddress());
 		}
 	}
};
// Register the BroadcastReceiver
IntentFilter filter = new IntentFilter(BluetoothDevice.ACTION_FOUND);
registerReceiver(mReceiver, filter); //别忘了在onDestory()时，取消注册。
{% endhighlight %}

>注意：对Bluetooth adapter来说设备搜寻操作是一个重量级的操作,这会消耗很多资源。一旦你发现一个要连接的设备，要在尝试连接之前通过`cancelDiscovery()`方法停止搜寻操作。如果你已经持有了一个设备连接，那么搜寻操作会减少通讯的可用带宽，因此，在连接时不要执行搜寻操作。


##<span id="enable_discoverability">开启可被发现功能</span>
如果你想让你的设备可被其他设备发现，调用`startActivityForResult(Intent, int)`方法，并传递一个带有action为` ACTION_REQUEST_DISCOVERABLE`的意图对象，这将通过系统设置发送一个可被发现的请求(并不会停止你的应用)。默认情况下，设备可被发现的时间会维持`120`秒，你可以通过为意图对象添加`EXTRA_DISCOVERABLE_DURATION`参数来定义此时间，允许的最大值为`3600`秒，`0`秒代表设备始终可以被发现。如果值小于0或者大于3600，将会按120秒处理，例如，下边的代码设置可被发现时间为300秒。

{% highlight java %}
Intent discoverableIntent = new
Intent(BluetoothAdapter.ACTION_REQUEST_DISCOVERABLE);
discoverableIntent.putExtra(BluetoothAdapter.EXTRA_DISCOVERABLE_DURATION, 300);
//startActivity(discoverableIntent);
//根据下文，应该是
startActivityForResult(discoverableIntent);
{% endhighlight %}

此时会出现一个请求用户开启可被发现选项的对话框。如果用户点击`确定`，会在onActivityResult()中，获得和请求的时间相等的结果码。如果用户选择了`取消`,那么结果码为`RESULT_CANCELED`。

><strong>注意：</strong>如果蓝牙没有开启的话，那么让设备可被发现的操作会自动开启蓝牙。

如果你想要知道发现模式改变的事件，你可以为`ACTION_SCAN_MODE_CHANGED`意图注册一个广播接受器。它包含`EXTRA_SCAN_MODE`和 `EXTRA_PREVIOUS_SCAN_MODE`,用于区分新的和旧的扫描模式。每个的可能值有`SCAN_MODE_CONNECTABLE_DISCOVERABLE`（可连接可被发现）, `SCAN_MODE_CONNECTABLE`（可连接不可被发现）, 或者`SCAN_MODE_NONE`（既不能连接也不能被发现）。 

如果你要连接一个远程设备，那么就不需要开启可被发现功能。此功能只有在你的应用开启一个server socket，接收连接时使用，因为在远程设备建立连接之前，必须发现你的设备。

##连接设备
为了在两个设备间建立连接，你必须实现服务端和客户端的机制。因为一个设备必须开启server socket，另一个设备必须初始化连接（使用服务端的MAC地址）。当服务端和客户端在同一RFCOMM通道内，都持有了一个连接的`BluetoothDevice`，那么认为连接就建立起来了。此时，每个设备都可以获得输入、输出流，并开始进行通讯。

服务端设备和客户端设备分别通过不同的方式获取`BluetoothSocket`。服务端会在接收连接请求时获取到它，客户端会在向服务端开启RFCOMM通道时获取到它。

一种实现方法是，每个设备都作为服务端，因此每一个都会开启一个server socket，并且监听连接请求。任何一个设备都可以与另一个建立连接，并成为客户端。另外，一个设备可以明确的按需开启一个server socket,其他的设备可以简单的初始化连接。

###作为服务端
两个设备要想连接，其中一个必须扮演服务端的角色，并持有一个开启的`BluetoothServerSocket`，server socket的目的是为了监听连接请求，并在连接时提供一个连接的`BluetoothSocket`。但从BluetoothServerSocket中取得BluetoothSocket之后，BluetoothServerSocket就可以（并应该）被丢弃，除非你想接受更多的设备连接。

以下是建立一个server socket并接受连接的基本过程。

<strong>1.通过调用`listenUsingRfcommWithServiceRecord(String, UUID)`方法获取`BluetoothServerSocket`。</strong>

第一个参数，是你的服务的可识别名称，系统会自动的将其写入一个新的服务发现协议（SDP-Service Discovery Protocol）数据库入口。（名称可以是任意的，可以简单的指定为你的程序名称）。第二个参数也包含在SDP入口内，并将成为和其他设备连接的协议的基础。即当客户端尝试与此设备连接时，客户端将会携带一个唯一的UUID，用于区分它想要连接的服务。为了能够连接，UUIDs必须匹配。

>关于UUID： Universally Unique Identifier通用唯一识别码，是一种标准化的128位格式字符串ID，用于唯一地标识信息。你可以使用在网上生产的随机的UUID用于你的应用程序，然后通过`String.fromString(String)`初始化UUID。

<strong>2.调用`accept()`方法开始监听连接请求</strong>

这个方法是阻塞式的，但接受连接请求或者发生异常时，就会退出。仅当一个远程的设备发起的连接请求的UUID与此注册的UUID匹配的时候，连接才会被接受。当成功的时候，`accept()`方法会返回一个`BluetoothSocket`。

<strong>3.除非想接受更多的连接，否则调用`close()`</strong>

`close()`方法会释放server socket以及所有的相关资源，但是不会切断通过accept()方法返回的BluetoothDevice的连接。和TCP/IP不同，RFCOMM在同一时间、同一信道内只允许有一个连接。因此，在大多数情况下，在接受了一个连接请求后，理解调用BluetoothServerSocket的close()方法是合理的。

不要在主线程中调用`accept()`方法，因此它是阻塞式的。通常我们会在新的线程中，完成对`BluetoothServerSocket`或者`BluetoothSocket`的操作，`BluetoothServerSocket`和`BluetoothSocket`中的所有方法都是线程安全的。

####举例
{% highlight java %}
private class AcceptThread extends Thread {
    private final BluetoothServerSocket mmServerSocket;
 
    public AcceptThread() {
        // Use a temporary object that is later assigned to mmServerSocket,
        // because mmServerSocket is final
        BluetoothServerSocket tmp = null;
        try {
            // MY_UUID 是应用的UUID字符串, 也会在客户端用到
            tmp = mBluetoothAdapter.listenUsingRfcommWithServiceRecord(NAME, MY_UUID);
        } catch (IOException e) { }
        mmServerSocket = tmp;
    }
 
    public void run() {
        BluetoothSocket socket = null;
        // 保持监听，直到异常发生或者socket返回。
        while (true) {
            try {
                socket = mmServerSocket.accept();
            } catch (IOException e) {
                break;
            }
            // 如果接受了一个连接
            if (socket != null) {
                // 管理连接(在一个单独的线程中)
                manageConnectedSocket(socket);
                mmServerSocket.close();
                break;
            }
        }
    }
 
    /** Will cancel the listening socket, and cause the thread to finish */
    public void cancel() {
        try {
            mmServerSocket.close();
        } catch (IOException e) { }
    }
}
{% endhighlight %}

>注意：当`accept()`返回`BluetoothSocket`时，socket已经连接了，所以不要调用`connect()`方法（客户端也一样）。

manageConnectedSocket()是一个虚构的方法，将会实例化一个用于传输数据的线程，将会在[管理一个连接](#)部分讨论。

在监听到连接后，通常要尽快的关闭`BluetoothServerSocket`，在上边的例子中，在获得`BluetoothSocket`之后就调用了`close()`方法。你可能也想在你的线程内部提供一个公共方法，当你需要停止对server socket的监听时，可以关闭私有的BluetoothSocket。

##<span id="Profiles">Profiles的使用</span>

##<span id="Manage_a_connection">管理一个连接</span>

##未完成……

