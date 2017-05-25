---
type: tutorial
layout: tutorial
title:  "Kotlin Android Extensions"
description: "This tutorial describes how to use Kotlin Android Extensions to improve support for Android development."
authors: Yan Zhulanow
showAuthorInfo: true
source:
---
在本章教程中，我们将逐步介绍如何使用Kotlin安卓扩展插件提升安卓的开发体验。

### 背景

相信每一位安卓开发人员对`findViewById()`这个方法再熟悉不过了，毫无疑问，潜在的bug和脏乱的代码是难以阅读和支持的。
尽管存在一系列的开源库能够为这个问题带来解决方案，然而对于运行时依赖的库，需要为每一个`View`注解变量字段。

现在Kotlin安卓扩展插件能够提供与这些开源库功能相同的体验，不需要添加任何额外代码，也不影响任何运行时体验。

因此，我们可以写出如下代码：

```kotlin
// Using R.layout.activity_main from the main source set
import kotlinx.android.synthetic.main.activity_main.*

class MyActivity : Activity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        textView.setText("Hello, world!")
        // Instead of findView(R.id.textView) as TextView
    }
}
```

`textView`是对`Activity`的一项扩展属性，与在`activity_main.xml`中的声明具有同样类型。

### 使用Kotlin安卓扩展

#### 依赖配置

{{ site.text_using_gradle }}

Android Extensions is a part of the Kotlin IDEA plugin. You do not need to install additional plugins.

All you need is to enable the Android Extensions Gradle plugin in your project-local `build.gradle` file:

``` groovy
apply plugin: 'kotlin-android-extensions'
```

#### Importing synthetic properties

It is convenient to import all widget properties for a specific layout in one go:

``` kotlin
import kotlinx.android.synthetic.main.<layout>.*
```

Thus if the layout filename is `activity_main.xml`, we'd import `kotlinx.android.synthetic.main.activity_main.*`.

If we want to call the synthetic properties on `View` (useful in adapter classes), we should also import `kotlinx.android.synthetic.main.activity_main.view.*`.

Once we do that, we can then invoke the corresponding extensions, which are properties named after the views in the XML file. 
For example, for this view:

``` xml
    <TextView
            android:id="@+id/hello"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="Hello World, MyActivity"
            />
```

There will be a property named `hello`:

``` kotlin
activity.hello.setText("Hi!")
```

### Android Flavors

Android Extensions plugin supports Android flavors. Suppose you have a flavor named `free` in your `build.gradle` file:

```
android {
    productFlavors {
        free {
            versionName "1.0-free"
        }
    }
}
```

So you can import all synthetic properties for the `free/res/layout/activity_free.xml` layout by adding this import:

```kotlin
import kotlinx.android.synthetic.free.activity_free.*
```

### Under the hood

Kotlin Android Extensions is a plugin for the Kotlin compiler, and it does two things:

1. Adds a hidden caching function and a field inside each Kotlin `Activity`. The method is pretty small so it doesn't increase the size of APK much.
2. Replaces each synthetic property call with a function call.

How this works is that when invoking a synthetic property, where the receiver is a Kotlin Activity/Fragment class that is in module sources, the caching function is invoked.
For instance, given

``` kotlin
class MyActivity: Activity()
fun MyActivity.a() { 
        this.textView.setText(“”) 
}
```

a hidden caching function is generated inside MyActivity, so we can use the caching mechanism.

However in the following case:

``` kotlin
fun Activity.b() { 
        this.textView.setText(“”)     
}
```

We wouldn't know if this function would be invoked on only Activities from our sources or on plain Java Activities also. As such, we don’t use caching there, even
if MyActivity instance from the previous example is the receiver.
