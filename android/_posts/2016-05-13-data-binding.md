---
layout: post
title: "Data Binding使用指南"
modified: 2016-05-13 19:50:57
excerpt: "Google官方推出的Data Binding库使用指南"
tags: [android, databinding]
published: true
---

<br>

**说明：本文是按照Android官方文档顺序进行部分翻译，并结合自身实践进行总结的，不保证100%还原官方内容，建议还是先看下官方的[说明文档](http://developer.android.com/intl/zh-cn/tools/data-binding/guide.html)**

这篇文章将教你如何使用Data Binding Library来书写声明式的(declarative)布局，以及使用尽可能少的代码来使应用逻辑与布局绑定。

Data Binding Library不仅灵活而且具有广泛的兼容性，它是个支持库，你可以应用到Android 2.1（API level 7+）之后的所有安卓平台上。

使用此支持库，需要使用1.5.0-alpha1或者更高版本的android gradle插件。


## 1 Build Environment - 构建环境
首先，我们需要从Android SDK Manager的Support repository中下载此库。

然后，在app module下的`build.gradle`文件中添加`dataBinding`元素。代码如下：

```
android {
    ....
    dataBinding {
        enabled = true
    }
}
```
<br>
如果你使用的支持库也使用了data binding，也需要在其`build。gradle`文件下进行同样的配置。

同时，要确保你使用的Android Studio是`1.3及以上的版本`。

> 注：添加以上配置之后，Sync一下，然后项目会自动添加依赖的库。

## 2 Data Binding Layout Files - Data Binding布局文件

### 2.1 Data Binding表达式

Data-binding布局文件稍有些不同，它的根布局标签为`layout`，包含一个`data`元素和`view`根元素，`view`元素就是我们正常使用的布局。举例如下：`activity_main.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

<br>
`data`内的`user`变量包含了可能在接下来的布局中被用到的属性。

```
<variable name="user" type="com.example.User"/>
```

<br>
在layout中的表达式，用`@{}`语句被写在相应的属性中。这里的TextView的text展示就是user的fristName属性值。

```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}"/>          
```

<br>
还可以做链式操作，`user.firstName`得到的`String`，我们可以继续调用`String`的相应方法

```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName.toUpperCase()}"/>          
```

<br>

#### <font color="#ff0000">遇到的坑</font>

注意视图的属性对不同参数类型的处理有没有区别，比如`android:text`，在Databinding内部应该是调用了TextView的`setText()`方法，如果`@{}`表达式内是数字的话，例如`@{user.age}`，会报资源找不到的错误（`android.content.res.Resources$NotFoundException`），因此我们的表达式应该是`@{String.valueOf(user.age)}`

### 2.2 Data Object - Data对象

我们创建一个在上边用到的数据对象

```java
public class User {
    private String firstName;
    private String lastName;

    public void setFirstName(String firstName) {
        this.firstName = firstName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }

    public String getLastName() {
        return lastName;
    }

    public String getFirstName() {
        return firstName;
    }
}
```

<br/>
在布局文件中，TextView的`android:text`属性，使用表达式`@{user.firstName}`将会访问User类的`firstName`属性以及`getFirstName()`方法，或者访问`firstName()`方法，如果它存在的话。

### 2.3 Binding Data - 绑定数据
默认情况下，Android Studio会自动根据以`layout`作为根布局的文件名称生产一个Binding类，比如上面的布局文件`activity_main`，生产的Binding类名称为`ActivityMainBinding`，然后在MainActivity里进行数据绑定：

```java
Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```

<br/>
ActivityMainBinding下的方法，都是根据布局文件中的`variable`标签的`name`属性自动生成的，因为我们的布局文件里有个name为user的variable，那么就生成了`setUser`方法，参数是`variable`type对应的类。

运行程序后，你就会在界面上看到文字Test User。或者，你可以通过以下方式获取：


```java
ActivityMainBinding binding = ActivityMainBinding.inflate(getLayoutInflater());
```

<br/>

<s>但实测，并没有什么卵用。官方文档没有描述清楚，这样明显是不行的，怎么跟视图关联的？这块至少要有个`setContentView(R.layout.activity_main)`吧？但是加上了也是不行。</s>

完整的代码应该是这样的：

```java
ActivityMainBinding binding = ActivityMainBinding.inflate(getLayoutInflater());
View view = binding.getRoot();
setContentView(view);
```

<br>

还有下面这种方式：

```java
View root = getLayoutInflater().inflate(R.layout.activity_main, null);
setContentView(root);
ActivityMainBinding binding = ActivityMainBinding.bind(root);
```

<br>

### 2.4 Binding Events - 事件绑定
理解了上边的数据绑定，事件绑定就好理解了，跟数据绑定类似。

以点击事件为例，声明一个variable，名称为`onClicklistener`，以`MainActivity`作为处理类

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">

    <data>

        <variable
            name="user"      type="com.chiemy.example.databindingexample.bean.User" />

        <variable
            name="onClicklistener"
type="com.chiemy.example.databindingexample.MainActivity"/>
    </data>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">


        <Button
            android:id="@+id/btn_list_item_binding"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="ListItemBinding"
            android:onClick="@{onClicklistener.onClick}"
            />
    </LinearLayout>

</layout>
```

<br/>
MainActivity下要实现布局文件表达式中用到的方法，接收参数为View：

```java
public void onClick(View view){
	//TODO 处理点击事件
}
```

<br>

#### <font color="#ff0000">遇到的坑</font>

使用Android Studio 2.1.1编译测试，本来开始用起来没有任何问题，但当我新建了一个Activity，再使用这种方式进行事件绑定时，问题出现了。在新的Activity的布局文件中也采用如上方式，点击按钮时应用直接崩溃了，提示我Activity里没有声明相应的方法（`java.lang.IllegalStateException: Could not find a method onClick(View) ......`），似乎对`android:onClick`表达式的识别出了问题，但在Android Studio 1.5.1上测试编译没有问题。如果你也遇到了同样的问题并找到了解决办法，请指点。


## 3 Layout Details - 布局深入
我们可以在`data`标签里使用`import`元素，这样我们可以像java一样，简单的导入一些类。

例如：

```xml
<data>
    <import type="android.view.View"/>
</data>
```

<br/>
现在我们就可以在binding表达式里使用View了

```xml
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isFriend ? View.VISIBLE : View.GONE}"/>
```

<br/>
实际测试(Android Studio 1.5.1)，在布局文件中这样使用会提示`Cannot resolve symbol`的错误，但是编译和运行并没有问题。

当类名有冲突的时候，我们可以使用`alias:`属性为类起个别名，比如有个类`com.example.real.estate.View`

```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

<br/>
现在，我们使用`Vista`引用的就是`com.example.real.estate.View`类，`View`引用的就是`android.view.View`类了。

在`variable`中，多次用到某个类的时候，`import`也是很有用处的。类似于java中，如果不导包，我们在每次用到某个类时，都要写类的全称（包名+类名），导包后我们只需写类名就可以了。

> 注：Android Studio还不支持类似java中的`import com.example.*`;

import的类型同时支持静态变量和方法的表达式：

```java
public class StringUtils {
    public static String capitalize(String text){
        return text.toUpperCase();
    }
}
```

<br/>

```xml
<data>
    <import type="com.chiemy.example.databindingexample.StringUtils"/>
</data>

<TextView
   android:text="@{StringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

<br/>

### 3.1 Variables
在`data`元素中可以有任意数量的`variable`元素，布局文件中的binding表达式可能会用到`variable`元素所描述的属性。

```xml
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```

<br/>

`variable`类型会在编译的时候被检查，如果它实现了`Observable`接口或者是一个`observabel collection`，应该反映到类型中。如果它是一个没有实现Observabled的基本的类或接口，它就不会被观察。

当对于不同配置（如，横竖布局）有不同的布局文件时，variables将会被合并，因此不同的布局之间不能存在冲突的variable定义。

生成的binding类，会为每个variable提供一个getter和setter方法，直到调用setter方法时，variable才会被设置Java的默认值，引用类型为null，int类型为0，boolean类型为false，等等。

有个默认的名为context的variable, 类型为Context, 它是通过根布局的getContext()方法得到的，我们可以直接使用

```java
public class StringUtils {
    public static String packageName(Context context){
        return context.getPackageName();
    }
}
```

<br/>

```xml
<TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{StringUtils.packageName(context)}"
            />
```

<br>

### 3.2 Custom Binding Class Names - 自定义Binding类的名称
默认情况下，Binding类的名称是根据类名生成的，去除布局名称中的“_”，以驼峰命名的形式，并以Binding结尾。这个类将被放置在module包下的databinding包下。例如，`contact_item.xml`将会生成`ContactItemBinding`，如果module的包为`com.example.my.app`，那么类所处的包为`com.example.my.app.databinding.`（但你是看不到的）。

通过`data`元素的`class`属性，Binding类可以被重命名或者指定所在的包，例如：

```xml
<data class="ContactItem">
    ...
</data>
```

<br/>
这个生成的Binding类名称为`ContactItem`，位于module包下的databinding包中。

如果我们想指定它直接在module包下，我们可以在前面加个`.`

```xml
<data class=".ContactItem">
    ...
</data>
```

<br/>
我们还可以指定其他包，但要注意包必须存在，不会自动生成。如我们的module包名为`com.example.app`, class可以是：

- com.example.ContactItem
- com.ContactItem

不能是不存在的包，如`com.other.ContactItem`。

### 3.3 Includes
Variable也可以传递到一个include的布局里：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```

<br/>
注意，要用到`xmlns:bind="http://schemas.android.com/apk/res-auto"`命名空间的声明。

同时，include的布局文件里，必须包含跟传递的variable相同的variable。

Data binding不支持include一个以merge元素作为直接孩子的布局，例如，下面的方式是不支持的：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge>
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```

<br/>

### 3.4 Expression Language - 表达式语言

#### Common Features - 通用属性
许多和Java表达式相同：

- 数学运算符 `+ - / * %`
- 字符连接 `+`
- 逻辑运算 `&& ||`
- 位运算符 `& | ^`
- 一元运算符 `+ - ! ~`
- 位移运算 `>> >>> <<`
- 比较 `== > < >= <=`
- instanceof
- Grouping ()
- Literals - character, String, numeric, null
- 转型
- 方法调用
- 属性访问
- 数组访问 `[ ]`
- 三目运算符

举例：

```xml
android:text="@{String.valueOf(index + 1)}"
android:visibility="@{age > 13 ? View.GONE : View.VISIBLE}"
android:transitionName='@{"image_" + id}'
```

<br/>

#### Missing Operations - 没有的操作

- this
- super
- new
- 显式泛型调用

#### Null Coalescing Operator - Null合并操作
选择不为空的值

```xml
android:text="@{user.displayName ?? user.lastName}"
```

<br/>

与以下三目运算等价

```
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

<br>

#### Avoiding NullPointerException - 空指针安全

生成的data binding代码自动检验null值，并避免空指针的发生。例如在`@{user.name}`表达式中，如果user是null的，user.name将会取默认值null，如果你引用user.age，age是int型，那么值将会是0。

#### Collections - 集合

通用的容器：数组、List、SparseArray、Map，可以通过`[ ]`方便的访问。

```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>
…
android:text="@{list[index]}"
…
android:text="@{sparse[index]}"
…
android:text="@{map[key]}"
```

<br>

#### String Literals - String迭代
当属性值用单引号包裹时，表达式内部用双引号。

```xml
android:text='@{map["firstName"]}'
```

<br>

也可以属性值用双引号包裹，表达式内使用`&quot;`或者反单引号(`)

```xml
android:text="@{map[`firstName`}"
android:text="@{map[&quot;firstName&quot;]}"
```

<br>

#### Resources - 资源
也可以在表达式中使用正常的语法访问资源：

```xml
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"
```

<br>

格式化的和复数的String，可以根据提供的参数进行匹配。

```xml
android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"
```

<br>

正常引用和表达式的对应关系如下：

|类型　　　　         |正常引用　　　|表达式引用   |
|:------------------|:----------|:-----------|
|String[]           |@array     |@stringArray|
|int[]              |@array     |@intArray|
|TypedArray         |@array     |@typedArray|
|Animator           |@animator |@animator|
|StateListAnimator	|@animator  |@stateListAnimator|
|color int          |@color     |@color|
|ColorStateList     |@color     |@colorStateList|


## 4 Data Objects - 数据对象
POJO可以用于data dinding，但是修改POJO并不会引起UI的更新。data binding的强大之处在于赋予你的数据对象当数据变化时去更新UI的能力。有三种不同的数据通知更新的机制，Observable objects, observable fileds，以及observable collections。


### 4.1 Observable Objects
实现Observable接口的类，允许监听器属性的变化。

`Observable`接口有添加和移除监听的能力，但是通知则依赖于开发者。为了使开发简单，`BaseObservable`类，已经实现了监听注册的机制。实现类还是得在属性变化的时候负责提醒。通过在getter方法上的`Bindable`注解实现监听，在setter方法中完成通知。

```java
private static class User extends BaseObservable {
   private String firstName;
   private String lastName;
   @Bindable
   public String getFirstName() {
       return this.firstName;
   }
   @Bindable
   public String getLastName() {
       return this.lastName;
   }
   public void setFirstName(String firstName) {
       this.firstName = firstName;
       notifyPropertyChanged(BR.firstName);
   }
   public void setLastName(String lastName) {
       this.lastName = lastName;
       notifyPropertyChanged(BR.lastName);
   }
}
```

<br>


### 4.2 ObservableFields

像上边的方式，我们有一部分工作花在了创建`Observable`类上，如果我们想节省时间，或者我们只有很少的属性，我们可以使用`ObservableField`，以及它的弟兄们- `ObservableBoolean`, `ObservableByte`, `ObservableChar`, `ObservableShort`, `ObservableInt`, `ObservableLong`, `ObservableFloat`, `ObservableDouble`, `ObservableParcelable`。`ObservableField`自己保存只有一个属性的observable对象，早期的版本在访问时会避免自动装箱和拆箱。使用方式如下：

```java
private static class User {
   public final ObservableField<String> firstName =
       new ObservableField<>();
   public final ObservableField<String> lastName =
       new ObservableField<>();
   public final ObservableInt age = new ObservableInt();
}
```

<br/>

设置和获取属性的时候用以下方式：

```java
user.firstName.set("Google");
int age = user.age.get();
```

<br/>

#### <font color="#ff0000">遇到的坑</font>

本想将`ObservableField`及相关的属性设置为私有的，然后简化getter方法，像下边这样：

```java
public class ObservableFiledsUser {
    private ObservableInt age = new ObservableInt();

    public void setAge(int age) {
        this.age.set(age);
    }

    public int getAge() {
        return age.get();
    }
}
```

<br>

但是这样做不会引起视图的自动更新，所以如果想将属性设置为私有的，那么getter方法一定要返回相应的类型，即：

```java
public ObservableInt getAge() {
	return age;
}
```
<br/>

### 4.3 Observable Collections
Data binding提供了具有通知功能的集合类，如`ObservableArrayMap`，`ObservableArrayList`，

`ObservableArrayMap`继承自`ArrayMap`，并实现了`ObservableMap`接口，使用方式和`Map`一样，只是内部实现具有自动的通知机制。

```java
ObservableArrayMap<String, Object> user = new ObservableArrayMap<>();
user.put("firstName", "Google");
user.put("lastName", "Inc.");
user.put("age", 17);
```

<br/>

```xml
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap&lt;String, Object>"/>
</data>
…
<TextView
   android:text='@{user["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

> 注意：variable的属性不能包含`<`符号，要用`&lt;`代替。


`ObservableArrayList`继承自`ArrayList`，并实现了`ObservableList`接口，使用方式和`List`一样，只是内部实现具有自动的通知机制。

```java
ObservableArrayList<Object> user = new ObservableArrayList<>();
user.add("Google");
user.add("Inc.");
user.add(17);
```

<br/>

```xml
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList&lt;Object>"/>
</data>
…
<TextView
   android:text='@{user[Fields.LAST_NAME]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

<br>

## 5 Generated Binding - Binding的生成

生成的Binding对象连接了layout变量及相关视图，像之前提到的，Binding对象的包及名称是可以自定义的，所有生成的Binding对象都继承自`ViewDataBinding`

### 5.1 Creating - 创建

创建方式，上边已经提到过，主要有以下几种方式：

使用Binding类的静态方法，有一个参数的版本和多个参数的版本：

```java
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater);
MyLayoutBinding binding = MyLayoutBinding.inflate(layoutInflater, viewGroup, false);
```
<br>

如果布局是用不同机制填充的，我们可以单独与layout进行绑定：

```java
MyLayoutBinding binding = MyLayoutBinding.bind(viewRoot);
```

<br>

有时Binding不能预知，我们可以使用`DataBindingUtil`类：

```java
ViewDataBinding binding = DataBindingUtil.inflate(LayoutInflater, layoutId,
    parent, attachToParent);
// 或
ViewDataBinding binding = DataBindingUtil.bindTo(viewRoot, layoutId);
```

<br>

### 5.2 Views With IDs - 带ID的视图

每个带有Id的视图，都会在binding类里生成一个对应的public final的字段，Binding在View层级上做一次遍历，取出所有带ID的视图，这种机制要比`findViewById`快，例如对于如下布局：

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
   android:id="@+id/firstName"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"
  android:id="@+id/lastName"/>
   </LinearLayout>
</layout>
```
<br>

最后生成的binding类里，就生成了如下字段：

```java
public final TextView firstName;
public final TextView lastName;
```
<br>

### 5.3 Variables - 变量

每个variable变量都会在Binding类里生成get和set方法，例如

```xml
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user"  type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note"  type="String"/>
</data>
```
<br>

会生成如下方法

```java
public abstract com.example.User getUser();
public abstract void setUser(com.example.User user);
public abstract Drawable getImage();
public abstract void setImage(Drawable image);
public abstract String getNote();
public abstract void setNote(String note);
```

<br>

### 5.4 ViewStubs

ViewStub和其他View类略有不同，它开始不可见，且当它可见或被填充时，它会把其他布局填充进来，把自己替换掉。

因为，ViewStub本质上在布局层级里是不存在的，因此只有在ViewStub.inflate()之后，才能进行数据绑定，我们可以使用`ViewStubProxy`进行操作。

```java
ViewStubActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_view_stub);
// 这样报转型错误？
// final ViewStubProxy viewStubProxy = new ViewStubProxy(binding.viewStub);
// 这样明显不对，但竟然能运行起来，结果也是正确的
// final ViewStubProxy viewStubProxy = binding.viewStub;
// 暂时采用这种方式
final ViewStubProxy viewStubProxy = new ViewStubProxy((ViewStub)findViewById(R.id.viewStub));
viewStubProxy.setOnInflateListener(new ViewStub.OnInflateListener() {
	@Override
	public void onInflate(ViewStub stub, View inflated) {
		InflatedLayoutBinding layoutBinding = (InflatedLayoutBinding)viewStubProxy.getBinding();
                // TODO 为InflatedLayoutBinding设置数据
    }
});
...
...
// 需要的时候，填充进来
if(!viewStubProxy.isInflated()){
	viewStubProxy.getViewStub().inflate();
}
```





