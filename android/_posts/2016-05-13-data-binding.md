---
layout: post
title: "Data Binding使用指南"
modified: 2016-05-13 19:50:57
excerpt: "Google官方推出的Data Binding库使用指南"
tags: [android, databinding]
published: true
---

这篇文章将教你如何使用Data Binding Library来书写声明式的(declarative)布局，以及使用尽可能少的代码来使应用逻辑与布局绑定。

Data Binding Library不仅灵活而且具有广泛的兼容性，它是个支持库，你可以应用到Android 2.1（API level 7+）之后的所有安卓平台上。

使用此支持库，需要使用1.5.0-alpha1或者更高版本的android gradle插件。


## 1 构建环境
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

## 2 Data Binding布局文件

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

`data`内的`user`变量包含了可能在接下来的布局中被用到的属性。

```
<variable name="user" type="com.example.User"/>
```

在layout中的表达式，用`@{}`语句被写在相应的属性中。这里的TextView的text展示就是user的fristName属性值。

```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}"/>          
```

还可以做链式操作，`user.firstName`得到的`String`，我们可以继续调用`String`的相应方法

```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName.toUpperCase()}"/>          
```


### 2.2 Data对象

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

在布局文件中，TextView的`android:text`属性，使用表达式`@{user.firstName}`将会访问User类的`firstName`属性以及`getFirstName()`方法，或者访问`firstName()`方法，如果它存在的话。

### 2.3 数据绑定
默认情况下，Android Studio会自动根据以`layout`作为根布局的文件名称生产一个Binding类，比如上面的布局文件`activity_main`，生产的Binding类名称为`ActivityMainBinding`，然后在MainActivity里进行数据绑定：

```java
Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   MainActivityBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```

MainActivityBinding下的方法，都是根据布局文件中的`variable`标签的`name`属性自动生成的，因为我们的布局文件里有个name为user的方法，那么就生成了`setUser`方法，参数是`variable`type对应的类。

运行程序后，你就会在界面上看到文字Test User。或者，你可以通过以下方式获取：


```java
MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());
```

但实测，并没有什么卵用。官方文档没有描述清楚，这样明显是不行的，怎么跟视图关联的？这块至少要有个`setContentView(R.layout.activity_main)`吧？但是加上了也是不行。

还有下面这种方式：

```java
View root = getLayoutInflater().inflate(R.layout.activity_main, null);
setContentView(root);
ActivityMainBinding binding = ActivityMainBinding.bind(root);
```

### 2.4 事件绑定
理解了上边的数据绑定，事件绑定久好理解了，跟数据绑定类似。

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

MainActivity下要实现布局文件表达式中用到的方法，接收参数为View：

```java
public void onClick(View view){
	//TODO 处理点击事件
}
```

## 3 布局深入
我们可以在`data`标签里使用`import`元素，这样我们可以像java一样，简单的导入一些类。

例如：

```xml
<data>
    <import type="android.view.View"/>
</data>
```

现在我们就可以在binding表达式里使用View了

```xml
<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isFriend ? View.VISIBLE : View.GONE}"/>
```

实际测试(Android Studio 1.5.1)，在布局文件中这样使用会提示`Cannot resolve symbol`的错误，但是编译和运行并没有问题。

当类名有冲突的时候，我们可以使用`alias:`属性为类起个别名，比如有个类`com.example.real.estate.View`

```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

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


```
<data>
    <import type="com.chiemy.example.databindingexample.StringUtils"/>
</data>

<TextView
   android:text="@{StringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

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

`variable`类型会在编译的时候被检查，如果它实现了`Observable`接口或者是一个`observabel collection`，应该反映到类型中。如果它是一个没有实现Observabled的基本的类或接口，它就不会被观察。

当对于不同配置（如，横竖布局）有不同的布局文件时，variables将会被合并，因此不同的布局直接不能存在冲突的variable定义。

生成的binding类，会为每个variable提供一个getter和setter方法，直到调用setter方法时，variable才会被设置Java的默认值，引用类型为null，int类型为0，boolean类型为false，等等。

有个默认的名为context的variable, 类型为Context, 它是通过根布局的getContext()方法得到的，我们可以直接使用

```java
public class StringUtils {
    public static String packageName(Context context){
        return context.getPackageName();
    }
}
```


```xml
<TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{StringUtils.packageName(context)}"
            />
```

### 3.2 自定义Binding类的名称
默认情况下，Binding类的名称是根据类名生成的，去除布局名称中的“_”，以驼峰命名的形式，并以Binding结尾。这个类将被放置在module包下的databinding包下。例如，`contact_item.xml`将会生成`ContactItemBinding`，如果module的包为`com.example.my.app`，那么类所处的包为`com.example.my.app.databinding.`（但你是看不到的）。

通过`data`元素的`class`属性，Binding类可以被重命名或者指定所在的包，例如：

```xml
<data class="ContactItem">
    ...
</data>
```

这个生成的Binding类名称为`ContactItem`，位于module包下的databinding包中。

如果我们想指定它直接在module包下，我们可以在前面加个`.`

```xml
<data class=".ContactItem">
    ...
</data>
```

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

### 3.4 表达式语言
#### 通用属性
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

#### 没有的操作：

- this
- super
- new
- 显式泛型调用

#### Null合并操作
选择不为空的值

```xml
android:text="@{user.displayName ?? user.lastName}"
```

与以下三目运算等价

```
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

#### 空指针安全

生成的data binding代码自动检验null值，并避免空指针的发生。例如在`@{user.name}`表达式中，如果user是null的，user.name将会取默认值null，如果你引用user.age，age是int型，那么值将会是0。

#### 集合

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

