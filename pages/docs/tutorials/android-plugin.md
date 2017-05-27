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

相信每一位安卓开发人员对`findViewById()`这个方法再熟悉不过了，毫无疑问，潜在的bug和脏乱的代码令后续开发无从下手的。
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

安卓扩展是Kotlin IDEA插件的组成之一，因此不需要再单独安装额外插件。

开发者仅需要在项目的`build.gradle`文件中启用Gradle安卓扩展插件即可：

``` groovy
apply plugin: 'kotlin-android-extensions'
```

#### 导入合成属性

仅需要一行即可非常方便导入指定布局文件中所有控件属性：

``` kotlin
import kotlinx.android.synthetic.main.<layout>.*
```

假设当前布局文件是`activity_main.xml`，我们只需要引入`kotlinx.android.synthetic.main.activity_main.*`。

若需要调用`View`的合成属性(在适配器类中非常有用)，同时还应该导入`kotlinx.android.synthetic.main.activity_main.view.*`。

导入完成后即可调用在xml文件中以视图控件命名属性的对应扩展，
比如下例:

``` xml
    <TextView
            android:id="@+id/hello"
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:text="Hello World, MyActivity"
            />
```

将有一个名为`hello`的属性：

``` kotlin
activity.hello.setText("Hi!")
```

### 安卓多渠道

安卓扩展插件现已支持安卓多渠道。假设当前在`build.gradle`文件中指定一个名为`free`的渠道：

```
android {
    productFlavors {
        free {
            versionName "1.0-free"
        }
    }
}
```

所以现在只需要添加一行导入语句即可从`free/res/layout/activity_free.xml`布局中导入所有的合成属性：

```kotlin
import kotlinx.android.synthetic.free.activity_free.*
```

### 缓存覆盖

Kotlin安卓扩展作为Kotlin编译器的插件，主要有两大作用：

1. 在每一个Kotlin`Activity`中添加一个隐藏缓存函数以及一个变量。该方法非常简洁，因此不会直接对APK体积有明显增加。
2. 使用函数调用替换每一个合成属性。

其工作原理是，当调用合成属性时，作为模块资源中Kotlin Activity/Fragment类的接收器，缓存函数被调用。
例如：

``` kotlin
class MyActivity: Activity()
fun MyActivity.a() { 
        this.textView.setText(“”) 
}
```

在MyActivity中生成一个隐藏缓存函数，因此我们可以使用缓存机制。 

然而在下面的例子中：

``` kotlin
fun Activity.b() { 
        this.textView.setText(“”)     
}
```

我们并不知道这个函数会被我们自己的来源或者普通的Java Activity调用，因此，即便在前一个示例中的MyActivity是接收器，也不会使用缓存。
