---
type: doc
layout: reference
category: "Syntax"
title: "委托属性"
---

# 委托属性

有一些常见的属性类型，虽然我们可以在每次需要的时候手动实现它们，
但是如果能够为大家把他们只实现一次并放入一个库会更好。例如包括：

* 延迟属性（lazy properties）: 其值只在第一次被访问的时候计算。
* 可观察属性（observable properties）: 属性变化时监听器收到通知，
* 把多个属性储存在一个映射（map）中，而不是每个存在单独的字段中。

为了支持这些（及其他）情况，Kotlin 支持 _委托属性_:

``` kotlin
class Example {
    var p: String by Delegate()
}
```

语法是： `val/var <属性名>: <类型> by <表达式>`。在 *by*{:.keyword} 后面的表达式是该 _委托_，
因为属性对应的 `get()`（和 `set()`）会被委托给它的 `getValue()` 和 `setValue()` 方法。
属性的委托不必实现任何的接口，但是需要提供一个 `getValue()` 函数（和 `setValue()`——对于 *var*{:.keyword} 属性）。
例如:

``` kotlin
class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name} in $thisRef.'")
    }
}
```

当我们从委托到一个 `Delegate` 实例的 `p` 读取时，将调用 `Delegate` 中的 `getValue()`，
所以它第一个参数是读出 `p` 的对象、第二个参数包含 `p` 自身的描述 
（例如你可以取他们的名字)。 例如:

``` kotlin
val e = Example()
println(e.p)
```

打印结果：

```
Example@33a17727, thank you for delegating ‘p’ to me!
```

类似地，当我们给 `p` 赋值, `setValue()` 函数就被调用. 前两个参数是一样的，第三个参数持有将要被赋予的值：

``` kotlin
e.p = "NEW"
```

打印结果：

```
NEW has been assigned to ‘p’ in Example@33a17727.
```

## 属性委托要求

这里我们总结委托对象的要求。

对于一个**只读**属性（即 *val*{:.keyword} 声明的），委托必须提供一个 `getValue`函数，该函数接受下面的参数：

* 接收者 —— 必须与 _属性所有者_ 类型相同或者是其父类（对于扩展属性——类型允许扩展），
* 元数据 —— 一定要是 `KProperty<*>` 的类型或它的父类型,

这个函数必须返回与属性相同的类型（或者对应的子类型）。

对于一个**可变**属性（即 *var*{:.keyword} 声明的），委托必须*额外*提供一个名为 `setValue` 的函数，该函数接受下面的参数：

* 接收者 —— 同 `getValue()`,
* 元数据 —— 同 `getValue()`,
* 新的值 —— 必须和属性同类型或者是它的父类型。

`getValue()` 或/和 `setValue()` 函数可以通过委托类的成员函数提供或者由扩展函数提供。
当你需要委托属性到原本未提供的这些函数的对象时后者会更便利。
两函数都需要用 `operator` 关键字来进行标记。


## 标准委托

Kotlin 标准库中对于一些有用的委托类型提供了工厂方法。

### 延迟属性 Lazy

函数 `lazy()` 接受一个 lambda 然后返回一个可以作为实现延迟属性的委托 `Lazy<T>` 的实例：
第一次调用 `get()` 会执行已传递给 `lazy()` 的 lamda 表达式并记录结果，
后续调用 `get()` 会直接返回记录的结果。


``` kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main(args: Array<String>) {
    println(lazyValue)
    println(lazyValue)
}
```

This example prints:

```
computed!
Hello
Hello
```

默认地，对于 lazy 属性的计算是**同步锁（synchronized）** 的：该值只在一个线程中计算，并且所有的线程
会看到相同的值。如果初始化代理的同步锁不是必需的，这样于多个线程
可以同时执行，那么将 `LazyThreadSafetyMode.PUBLICATION` 作为一个参数传递给 `lazy()` 函数。
而且如果你确定初始化将总是发生在单个线程，那么你可以使用 `LazyThreadSafetyMode.NONE` 模式，
它不会有任何线程安全的保证和相关的开销。


### 可观察属性 Observable

`Delegates.observable()` 接受两个参数：初始值和修改时处理程序（handler）。
在每次我们给属性赋值时会调用该处理程序（在赋值*后*执行）。它有三个
参数：被赋值的属性、旧的值和新的值：

``` kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main(args: Array<String>) {
    val user = User()
    user.name = "first"
    user.name = "second"
}
```

这个例子输出：

```
<no name> -> first
first -> second
```

如果你想能够打断和“否决”它，就使用 `vetoable()` 取代 `observable()`。
在属性被赋新值*之前*会执行传递给 `vetoable` 的处理程序。

## 把属性储存在映射（Map）中

一个常见的用例是在一个映射里存储属性的值。
这经常出现在像解析 JSON 或者做其他“动态”的事情的应用中。
在这中情况下，你可以使用 map 实例自身作为代理来代理属性。

``` kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

在这个例子中，构造函数会接收一个 map 参数：

``` kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

委托属性会从这个 map 中取值（通过 string 类型的 key——属性的名字）：


``` kotlin
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```

这对 *var*{:.keyword} 属性也奏效，只要把只读的 `Map` 换成 `MutableMap`：

``` kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```
