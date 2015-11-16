---
layout: post
title: "Android Wear应用布局的创建"
modified: 2015-11-11 14:51:03
excerpt: "本文将介绍如何创建Android wear布局及一些基本控件的使用"
tags: [android, android wear]
published: true
---
搭载Android Wear的手表现在已经很多了，屏幕有方的，有圆的。如何让界面在方形与圆形的设备上完美的展示，我想我们多少都有些困惑。还好，官方已经给出了比较完善的解决方案。

Android的一些基本控件我们是可以使用的，为了更好的适应Android Wear设备，还有些Android wear特有的控件，我们需要先了解一下。这些控件都在`com.google.android.support:wearable:+`支持包中。

###WatchViewStub

此控件用来为圆形、方形屏幕指定不同的布局。它是在运行时对屏幕的形状进行检测的，然后根据不同的屏幕形状填充不同的布局。

####用法：

首先我们将WatchViewStub作为Activity的根布局

	<android.support.wearable.view.WatchViewStub
    	xmlns:android="http://schemas.android.com/apk/res/android"
    	xmlns:app="http://schemas.android.com/apk/res-auto"
    	xmlns:tools="http://schemas.android.com/tools"
    	android:id="@+id/watch_view_stub"
    	android:layout_width="match_parent"
    	android:layout_height="match_parent"
    	app:rectLayout="@layout/rect_activity_wear"
    	app:roundLayout="@layout/round_activity_wear">
	</android.support.wearable.view.WatchViewStub>
	
其中，`app:rectLayout`属性用来指定方形屏幕所引用的布局文件，`app:roundLayout`用于指定圆形屏幕所引用的布局文件。

由于是运行时填充的，那么布局里包含的控件我们需要在填充完毕后才能获取：

	@Override
	protected void onCreate(Bundle savedInstanceState) {
    	super.onCreate(savedInstanceState);
    	setContentView(R.layout.activity_wear);

    	WatchViewStub stub = (WatchViewStub) findViewById(R.id.watch_view_stub);
    	stub.setOnLayoutInflatedListener(new 			WatchViewStub.OnLayoutInflatedListener() {
        		@Override 
        		public void onLayoutInflated(WatchViewStub stub) {
            		// 在这里我们可以获取相应的控件了
        		}
    	});
	}

###BoxInsetLayout
能够根据屏幕形状，进行自动调整的布局控件，使得在圆形屏幕下能够完整的显示布局。

我们通过为其子布局指定`app:layout_box`属性来指定显示区域，可能的取值及布局的位置：

<img src="http://7o4zgd.com1.z0.glb.clouddn.com/uilib02.png" width="200"/>

- **<font color="#ff0000">top</font>** 图中，红线之下的区域

- **<font color="#00ff00">bottom</font>** 图中，绿线之上的区域

- **<font color="#0000ff">right</font>** 图中，蓝线右侧的区域

- **<font color="#FFD700">left</font>** 图中，黄线左侧的区域

- **all** 中间灰色区域

以上属性可组合使用。

使用示例：

{% highlight java %}
<android.support.wearable.view.BoxInsetLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

   	<FrameLayout
      	android:layout_width="match_parent"       		android:layout_height="match_parent"
       app:layout_box="all">

       <TextView
       	android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:background="#ff0000"
          android:text="@string/hello_round" />

    </FrameLayout>
</android.support.wearable.view.BoxInsetLayout>
{% endhighlight %}

###WearableListView
此控件是Android wear上的ListView，继承自`android.support.v7.widget.RecyclerView`

####用法
- 创建自定义适配器

WearableListView的`setApdater()`方法接收的参数是`android.support.v7.widget.RecyclerView.Adapter`，WearableListView同时提供了一个继承自此Apdater的子类`WearableListView.Adapter`，我们可以选择继承此适配器来创建一个自定义适配器。

{% highlight java %}
private static class MyAdapter extends WearableListView.Adapter {
        private String[] mDataset;
        private final Context mContext;
        private final LayoutInflater mInflater;

        // Provide a suitable constructor (depends on the kind of dataset)
        public MyAdapter(Context context, String[] dataset) {
            mContext = context;
            mInflater = LayoutInflater.from(context);
            mDataset = dataset;
        }

        // Provide a reference to the type of views you're using
        public static class ItemViewHolder extends WearableListView.ViewHolder {
            private TextView textView;

            public ItemViewHolder(View itemView) {
                super(itemView);
                // find the text view within the custom item's layout
                textView = (TextView) itemView.findViewById(R.id.name);
            }
        }

        // Create new views for list items
        // (invoked by the WearableListView's layout manager)
        @Override
        public WearableListView.ViewHolder onCreateViewHolder(ViewGroup parent,
                                                              int viewType) {
            // Inflate our custom layout for list items
            return new ItemViewHolder(mInflater.inflate(R.layout.layout_wearable_list_item, null));
        }

        @Override
        public void onBindViewHolder(WearableListView.ViewHolder holder, int position) {
            // retrieve the text view
            ItemViewHolder itemHolder = (ItemViewHolder) holder;
            TextView view = itemHolder.textView;
            // replace text contents
            view.setText(mDataset[position]);
            // replace list item's metadata
            holder.itemView.setTag(position);
        }

        @Override
        public int getItemCount() {
            return mDataset.length;
        }


    }
{% endhighlight %}


- 创建带动画效果的Item视图

WearableListView可以通过很简单的方式为item实现滑动时的动画效果，只需要让自定义视图实现`WearableListView.OnCenterProximityListener`接口，然后在`onCenterPosition()`和`onNonCenterPosition()`回调方法中对动画进行管理即可，示例如下：

{% highlight java %}
public class WearableListItemLayout extends LinearLayout implements WearableListView.OnCenterProximityListener{
    private ImageView iconIv;
    private TextView nameTv;
    private int mFadedCircleColor, mChosenCircleColor;
    public WearableListItemLayout(Context context) {
        this(context, null);
    }

    public WearableListItemLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public WearableListItemLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mFadedCircleColor = getResources().getColor(R.color.grey);
        mChosenCircleColor = getResources().getColor(R.color.blue);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        iconIv = (ImageView) findViewById(R.id.circle);
        nameTv = (TextView) findViewById(R.id.name);
    }

    @Override
    public void onCenterPosition(boolean b) {
        nameTv.setAlpha(1f);
        ((GradientDrawable) iconIv.getDrawable()).setColor(mChosenCircleColor);
    }

    @Override
    public void onNonCenterPosition(boolean b) {
        ((GradientDrawable) iconIv.getDrawable()).setColor(mFadedCircleColor);
        nameTv.setAlpha(0.4f);
    }
}
{% endhighlight %}


- Item点击监听


WearableListView并没有提供直接设置Item点击监听的方法，需要通过其`setClickListener(WearableListView.ClickListener listener)`方法间接实现，如下：

{% highlight java %}
wearableLv.setClickListener(new WearableListView.ClickListener() {
                    @Override
                    public void onClick(WearableListView.ViewHolder viewHolder){
                        Integer position = (Integer) viewHolder.itemView.getTag();
                        // position 点击的位置
                    }

                    @Override
                    public void onTopEmptyRegionClick() {

                    }
                });
{% endhighlight %}

	

