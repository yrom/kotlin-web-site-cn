---
type: tutorial
layout: tutorial
title:  "Android Frameworks Using Annotation Processing"
description: "This tutorial describes how to use in Kotlin popular Android frameworks and libraries that rely on annotation processing."
authors: Svetlana Isakova
showAuthorInfo: false
source:
---

在日常Android开发中，流行着数以千计的框架帮助我们提升开发效率。使用Kotlin开发时仍然可以使用这些框架，而且和Java同样简单。本章教程将提供相关示例并重点突出配置的差异。

教程以[Dagger](android-frameworks.html#dagger), [Butterknife](android-frameworks.html#butterknife), [Data Binding](android-frameworks.html#data-binding), [Auto-parcel](android-frameworks.html#auto-parcel)以及[DBFlow](android-frameworks.html#dbflow)进行示例(其余框架配置基本类似)。
以上框架均以注解处理方式工作：通过对代码注解后自动生成模板代码。
注解有助于减少冗余代码，让代码清晰可读，想要了解运行时的代码，可以直接阅读自动生成的源代码。
但所有生成的代码均是Java代码而非Kotlin。

在Kotlin中添加依赖与Java类似，需要使用[Kotlin Annotation processing tool](/docs/reference/kapt.html) (`kapt`)代替`annotationProcessor`。


### Dagger

[Dagger](https://google.github.io/dagger//) 是著名的依赖注入框架。
如果你对它还不了解，可以查阅[用户手册](https://google.github.io/dagger//users-guide.html)。
我们已经将整个[(咖啡示例)the coffee example](https://github.com/google/dagger/tree/master/examples/simple) 
使用Kotlin重写，详细代码在[这里](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/kotlin-dagger)。 
Kotlin代码与Java非常相似；所有示例代码可在同一[文件](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/kotlin-dagger/src/main/kotlin/Coffee.kt)内查看。

和Java一样，Dagger使用`@Inject`对构造函数注解，从而创建类的实例。
Kotlin拥有更简洁的语法同时声明属性和构造函数。
要对构造函数进行注解，必须显示使用`constructor`关键字，并在关键字前声明`@Inject`。

```kotlin
class Thermosiphon 
@Inject constructor(
        private val heater: Heater
) : Pump {
    // ...
}    
```

注解方法看上去完全相同。
在下面的示例中，`@Binds`决定了无论何时需要`Pump`，使用都是`Thermosiphon`对象，`@Provides`指定了`Heater`的构造方式，`@Singleton`则表示`Heater`是全局单例:

```kotlin
@Module
abstract class PumpModule {
    @Binds
    abstract fun providePump(pump: Thermosiphon): Pump
}

@Module(includes = arrayOf(PumpModule::class))
class DripCoffeeModule {
    @Provides @Singleton
    fun provideHeater(): Heater = ElectricHeater()
}
```

`@Module`-对类注解，定义如何提供不同对象.
需要注意的是，作为多参数传递注解参数时时，需要显示的使用`arrayOf`进行包装，比如上文示例`@Module(includes = arrayOf(PumpModule::class))`。

使用`@Component`为类型生成依赖注入的实现。自动生成类的类名带有Dagger前置，比如`DaggerCoffeeShop`：

```kotlin
@Singleton
@Component(modules = arrayOf(DripCoffeeModule::class))
interface CoffeeShop {
    fun maker(): CoffeeMaker
}

fun main(args: Array<String>) {
    val coffee = DaggerCoffeeShop.builder().build()
    coffee.maker().brew()
}
``` 

Dagger为`CoffeeShop`生成的一套实现能够让你获取一个完全注入的`CoffeeMaker`。
`DaggerCoffeeShop`的具体代码实现可在IDE中查看。

我们注意到转换到Kotlin时代码几乎没有发生改变。
现在我们看看构造脚本(build script)中有些什么变化。  

在Java中需要指定`Dagger`作为`annotationProcessor `(或`apt`)依赖：

``` groovy
dependencies {
  ...
  annotationProcessor "com.google.dagger:dagger-compiler:$dagger-version"
}
```

在Kotlin中则需要添加`kotlin-kapt`插件激活`kapt`，并使用`kapt`替换`annotationProcessor`：

``` groovy
apply plugin: 'kotlin-kapt'
dependencies {
    ...
    kapt "com.google.dagger:dagger-compiler:$dagger-version"
}
```

特别提示：`kapt`也能够处理Java文件，所以不需要再保留`annotationProcessor`的依赖。

查看示例项目的完整[构建脚本](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/kotlin-dagger/build.gradle)，
以及[Android示例](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-dagger)转换后的代码。


### ButterKnife

[ButterKnife](http://jakewharton.github.io/butterknife/) allows to bind views to fields directly instead of calling `findViewById`. 

Note that [Kotlin Android Extensions](https://kotlinlang.org/docs/tutorials/android-plugin.html) plugin (automatically bundled into the Kotlin plugin in Android Studio) solves the same issue: replacing `findViewById` with a concise and straightforward code.
Consider using it unless you're already using ButterKnife and don't want to migrate.
 
You use `ButterKnife` with Kotlin in the same way as you use it with Java.
Let's see first the changes in the Gradle build script, and then highlight some of the differences in the code.
 
In the Gradle dependencies you use add the `kotlin-kapt` plugin and replace `annotationProcessor` with `kapt`:

``` groovy
apply plugin: 'kotlin-kapt'

dependencies {
    ...
    compile "com.jakewharton:butterknife:$butterknife-version"
    kapt "com.jakewharton:butterknife-compiler:$butterknife-version"
}
```

We've converted the ButterKnife [sample](https://github.com/JakeWharton/butterknife/tree/master/sample/app/src/main/java/com/example) to Kotlin.
The resulting code can be found [here](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-butterknife).

Let's look over it to spot what has changed.
In Java you annotated the field, binding it with the corresponding view:
 
``` java 
@BindView(R2.id.title) TextView title;
```

In Kotlin you can't work with fields directly, you work with [properties](/docs/reference/properties.html). 
You annotate the property:

``` kotlin
@BindView(R2.id.title)
lateinit var title: TextView
```
The `@BindView` annotation is defined to be applied to the fields only, but the Kotlin compiler understands that and annotates the corresponding field under the hood when you apply the annotation to the whole property.

Note how [the lateinit modifier](/docs/reference/properties.html#late-initialized-properties) allows to declare a non-null type initialized after the object is created (after the constructor call).
Without `lateinit` you'd have to declare a [nullable type](/docs/reference/null-safety.html) and add additional nullability checks.
 
You can also configure methods as listeners, using ButterKnife annotations:

``` java
@OnClick(R2.id.hello)
internal fun sayHello() {
    Toast.makeText(this, "Hello, views!", LENGTH_SHORT).show()
}
```

This code specifies an action to be performed on the "hello" button click.
Note that with lambdas this code looks rather concise written directly in Kotlin:

``` kotlin
hello.setOnClickListener {
    toast("Hello, views!")
}
```

The `toast` function is defined in the [Anko](https://github.com/Kotlin/anko) library.

### Data Binding

The [Data Binding Library](https://developer.android.com/topic/libraries/data-binding/index.html) allows you to bind your application data to the layouts in a concise way.

You enable the library using the same configuration as in Java:

``` groovy
android {
    ...
    dataBinding {
        enabled = true
    }
}
```

To make it work with Kotlin classes add the `kapt` dependency: 

``` groovy
apply plugin: 'kotlin-kapt'
dependencies {
    kapt "com.android.databinding:compiler:$android_plugin_version"
}  
```

When you switch to Kotlin, your xml layout files don't change at all.
For instance, you use `variable` within `data` to describe a variable that may be used within the layout.
You can declare a variable of a Kotlin type:
 
```xml
<data>
    <variable name="data" type="org.example.kotlin.databinding.WeatherData"/>
</data>
``` 

You use the `@{}` syntax for writing expressions, which can now refer Kotlin [properties](/docs/reference/properties.html): 

```xml
<ImageView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:src="@{data.imageUrl}"
    android:contentDescription="@string/image" />
```

Note that the databinding expression language uses the same syntax for referring to properties as Kotlin: `data.imageUrl`.
In Kotlin you can write `v.prop` instead of `v.getProp()` even if `getProp()` is a Java method.
Similarly, instead of calling a setter directly, you may use an assignment:
  
```kotlin
class MainActivity : AppCompatActivity() {
    // ...
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val binding: ActivityMainBinding =
                DataBindingUtil.setContentView(this, R.layout.activity_main)

        binding.data = weather
        // the same as
        // binding.setData(weather)
    }
}
```

You can bind a listener to run an action when a specific event happens:

```xml
<Button
    android:text="@string/next"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:onClick="startOtherActivity" />
```

Here `startOtherActivity` is a method defined in our `MainActivity`:

```kotlin
class MainActivity : AppCompatActivity() {
    // ...
    fun startOtherActivity(view: View) = startActivity<OtherActivity>()
}
```

This example uses the utility function `startActivity` creating an intent with no data and starting a new activity, which comes from the [Anko](https://github.com/Kotlin/anko) library.
To pass some data, you can say `startActivity<OtherActivity>("KEY" to "VALUE")`.

Note that instead of declaring lambdas in xml like in the following example, you can can bind actions directly in the code: 

```xml
<Button 
    android:layout_width="wrap_content" 
    android:layout_height="wrap_content"
    android:onClick="@{() -> presenter.onSaveClick(task)}" />
```          

``` kotlin
// the same logic written in Kotlin code
button.setOnClickListener { presenter.onSaveClick(task) }
```

In the last line `button` is referenced by `id` using the [Kotlin Android Extensions](https://kotlinlang.org/docs/tutorials/android-plugin.html) plugin. 
Consider using this plugin as an alternative which allows you to keep binding logic in code and have the concise syntax at the same time.    

You can find an example project [here](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-databinding).

### DBFlow

[DBFlow](https://github.com/Raizlabs/DBFlow) is a SQLite library that simplifies interaction with databases.
It heavily relies on annotation processing.

To use it with Kotlin configure annotation processing dependency using `kapt`:

``` kotlin
apply plugin: 'kotlin-kapt'

kapt "com.github.raizlabs.dbflow:dbflow-processor:$dbflow_version"
compile "com.github.raizlabs.dbflow:dbflow-core:$dbflow_version"
compile "com.github.raizlabs.dbflow:dbflow:$dbflow_version"
compile "com.github.raizlabs.dbflow:dbflow-kotlinextensions:$dbflow_version"
```

[Here](https://agrosner.gitbooks.io/dbflow/content/including-in-project.html) is a detailed guide how to configure DBFlow.

If your application already uses DBFlow, you can safely introduce Kotlin into your project. 
You can gradually convert existing code to Kotlin (ensuring that everything compiles along the way).
The converted code doesn't differ much from Java. 
For instance, declaring a table looks similar to Java with the small difference that default values for properties must be specified explicitly:
 
``` kotlin 
@Table(name="users", database = AppDatabase::class)
class User: BaseModel() {

    @PrimaryKey(autoincrement = true)
    @Column(name = "id")
    var id: Long = 0

    @Column
    var name: String? = null
}
``` 

Besides converting existing functionality to Kotlin, you can also enjoy the Kotlin specific module.
DBFlow defines a bunch of extensions to make its usage in Kotlin more idiomatic.
Let's highlight some of the supported features.

You can declare tables as data classes:

``` kotlin
@Table(database = KotlinDatabase::class)
data class User(@PrimaryKey var id: Long = 0, @Column var name: String? = null)
```

You can express queries via C#-like LINQ syntax.
Thus the Java code below can be either converted directly or rewritten into the following style:

``` java
/* java */
List<Result> = SQLite.select()
    .from(Result.class)
    .where(Result_Table.column.eq(6))
    .and(Result_Table.column2.in("5", "6", "9")).queryList()
```  
                    
``` kotlin 
/* kotlin */                   
val results = (select
      from Result::class
      where (column eq 6)
      and (column2 `in`("5", "6", "9"))
      groupBy column).list
```
 
Lambdas allow to write much simpler code for asynchronous computations:

``` kotlin
var items = (select from TestModel::class).list

// delete all these items.
items.processInTransactionAsync { it, databaseWrapper -> it.delete(databaseWrapper) }
``` 

More details can be found [here](https://agrosner.gitbooks.io/dbflow/content/KotlinSupport.html).
You can also browse the converted [sample application](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-dbflow).



### Auto-Parcel

[Auto-Parcel](https://github.com/frankiesardo/auto-parcel) allows to generate `Parcelable` values for classes annotated with `@AutoValue`.

When you specify the dependency you again use `kapt` as annotation processor to take care of Kotlin files: 
 
``` groovy
apply plugin: 'kotlin-kapt'

dependencies {
    ...
    kapt "frankiesardo:auto-parcel:$latest-version"
}
```

The converted [sample](https://github.com/frankiesardo/auto-parcel/tree/master/sample) can be found [here](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/android-auto-parcel).

You can annotate Kotlin classes with `@AutoValue`.
Let's look at the converted [`Address`](https://github.com/frankiesardo/auto-parcel/blob/master/sample/src/main/java/model2/Address.java) class for which the `Parcelable` implementation will be generated:

``` kotlin
@AutoValue
abstract class Address : Parcelable {
    abstract fun coordinates(): DoubleArray
    abstract fun cityName(): String

    companion object {
        fun create(coordinates: DoubleArray, cityName: String): Address {
            return builder().coordinates(coordinates).cityName(cityName).build()
        }
        
        fun builder(): Builder = `$AutoValue_Address`.Builder()
    }
    
    @AutoValue.Builder
    interface Builder {
        fun coordinates(x: DoubleArray): Builder
        fun cityName(x: String): Builder
        fun build(): Address
    }
}
```

Kotlin doesn't have `static` methods, so they should be place inside a [`companion object`](/docs/reference/object-declarations.html#companion-objects).
If you still want to use them from Java code, annotate them with [`@JvmStatic`](/docs/reference/java-to-kotlin-interop.html#static-methods).

If you need to access a Java class or method with a name that is not a valid identifier in Kotlin, you can [escape the name](/docs/reference/java-interop.html#escaping-for-java-identifiers-that-are-keywords-in-kotlin) with the  backtick (\`) character, like in accessing the generated class \``$AutoValue_Address`\`.
  
Overall the converted code looks very similar to the original Java code.