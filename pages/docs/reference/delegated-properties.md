---
type: doc
layout: reference
category: "Syntax"
title: "委托属性"
---

# 委托属性

有一些常见的属性类型，虽然我们可以在每次需要的时候手动实现它们，
但是如果能够为大家把他们只实现一次并放入一个库会更好。例如包括

* 延迟属性（lazy properties）: 其值只在首次访问时计算，
* 可观察属性（observable properties）: 监听器会收到有关此属性变更的通知，
* 把多个属性储存在一个映射（map）中，而不是每个存在单独的字段中。

为了涵盖这些（以及其他）情况，Kotlin 支持 _委托属性_:

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

当我们从委托到一个 `Delegate` 实例的 `p` 读取时，将调用 `Delegate` 中的 `getValue()` 函数，
所以它第一个参数是读出 `p` 的对象、第二个参数保存了对 `p` 自身的描述 
（例如你可以取它的名字)。 例如:

``` kotlin
val e = Example()
println(e.p)
```

输出结果：

```
Example@33a17727, thank you for delegating ‘p’ to me!
```

类似地，当我们给 `p` 赋值时，将调用 `setValue()` 函数。前两个参数相同，第三个参数保存将要被赋予的值：

``` kotlin
e.p = "NEW"
```

输出结果：

```
NEW has been assigned to ‘p’ in Example@33a17727.
```

## 属性委托要求

这里我们总结了委托对象的要求。

对于一个**只读**属性（即 *val*{:.keyword} 声明的），委托必须提供一个名为 `getValue` 的函数，该函数接受以下参数：

* 接收者 —— 必须与 _属性所有者_ 类型（对于扩展属性——指被扩展的类型）相同或者是它的超类型，
* 元数据 —— 必须是类型 `KProperty<*>` 或其超类型,

这个函数必须返回与属性相同的类型（或其子类型）。

对于一个**可变**属性（即 *var*{:.keyword} 声明的），委托必须*额外*提供一个名为 `setValue` 的函数，该函数接受以下参数：

* 接收者 —— 同 `getValue()`,
* 元数据 —— 同 `getValue()`,
* 新的值 —— 必须和属性同类型或者是它的超类型。

`getValue()` 或/和 `setValue()` 函数可以通过委托类的成员函数提供或者由扩展函数提供。
当你需要委托属性到原本未提供的这些函数的对象时后者会更便利。
两函数都需要用 `operator` 关键字来进行标记。


## 标准委托

Kotlin 标准库为几种有用的委托提供了工厂方法。

### 延迟属性 Lazy

`lazy()` 是接受一个 lambda 并返回一个 `Lazy <T>` 实例的函数，返回的实例可以作为实现延迟属性的委托：
第一次调用 `get()` 会执行已传递给 `lazy()` 的 lamda 表达式并记录结果，
后续调用 `get()` 只是返回记录的结果。


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

默认情况下，对于 lazy 属性的求值是**同步锁的（synchronized）**：该值只在一个线程中计算，并且所有线程
会看到相同的值。如果初始化委托的同步锁不是必需的，这样多个线程
可以同时执行，那么将 `LazyThreadSafetyMode.PUBLICATION` 作为参数传递给 `lazy()` 函数。
而如果你确定初始化将总是发生在单个线程，那么你可以使用 `LazyThreadSafetyMode.NONE` 模式，
它不会有任何线程安全的保证和相关的开销。


### 可观察属性 Observable

`Delegates.observable()` 接受两个参数：初始值和修改时处理程序（handler）。
每当我们给属性赋值时会调用该处理程序（在赋值*后*执行）。它有三个
参数：被赋值的属性、旧值和新值：

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

如果你想能够截获一个赋值并“否决”它，就使用 `vetoable()` 取代 `observable()`。
在属性被赋新值生效*之前*会调用传递给 `vetoable` 的处理程序。

## 把属性储存在映射中

一个常见的用例是在一个映射（map）里存储属性的值。
这经常出现在像解析 JSON 或者做其他“动态”事情的应用中。
在这种情况下，你可以使用映射实例自身作为委托来实现委托属性。

``` kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}
```

在这个例子中，构造函数接受一个映射参数：

``` kotlin
val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))
```

委托属性会从这个映射中取值（通过字符串键——属性的名称）：


``` kotlin
println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```

这也适用于 *var*{:.keyword} 属性，如果把只读的 `Map` 换成 `MutableMap` 的话：

``` kotlin
class MutableUser(val map: MutableMap<String, Any?>) {
    var name: String by map
    var age: Int     by map
}
```
