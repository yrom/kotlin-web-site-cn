---
type: doc
layout: reference
title: "Kotlin 1.1 的新特性"
---

# Kotlin 1.1 的新特性

## Table of Contents

* [协程](#协程（实验性的）)
* [其他语言功能](#其他语言功能)
* [标准库](#标准库)
* [JVM 后端](#jvm-后端)
* [JavaScript 后端](#javascript-后端)

## JavaScript

从 Kotlin 1.1 开始，JavaScript 目标平台不再当是实验性的。所有语言功能都支持，
并且有许多新的工具用于与前端开发环境集成。更详细改动列表，请参见[下文](#javascript-后端)
。

## 协程（实验性的）

Kotlin 1.1 的关键新特性是*协程*，它带来了 `future`/`await`、 `yield` 以及类似的编程模式的
支持。Kotlin 的设计中的关键特性是协程执行的实现是语言库的一部分，
而不是语言的一部分，所以你不必绑定任何特定的编程范式或并发库。

协程实际上是一个轻量级的线程，可以暂停并稍后恢复。Coroutines are supported through [*suspending functions*](coroutines.html#挂起函数): a call to such a function can potentially suspend a coroutine, and to start a new coroutine we usually use an anonymous suspending functions (i.e. suspending lambdas).

Let's look at `async`/`await` which is implemented in an external library, [kotlinx.coroutines](https://github.com/kotlin/kotlinx.coroutines): 

``` kotlin
// runs the code in the background thread pool
fun asyncOverlay() = async(CommonPool) {
    // start two async operations
    val original = asyncLoadImage("original")
    val overlay = asyncLoadImage("overlay")
    // and then apply overlay to both results
    applyOverlay(original.await(), overlay.await())
}

// launches new coroutine in UI context
launch(UI) {
    // wait for async overlay to complete
    val image = asyncOverlay().await()
    // and then show it in UI
    showImage(image)
}
```

Here, `async { ... }` starts a coroutine and, when we use `await()`, the execution of the coroutine is suspended while the operation being awaited is executed, and is resumed (possibly on a different thread) when the operation being awaited completes.

标准库通过 `yield` 和 `yieldAll` 函数使用协程来支持*惰性生成序列*。
在这样的序列中，在取回每个元素之后暂停返回序列元素的代码块，
并在请求下一个元素时恢复。这里有一个例子：

<div class="sample" markdown="1" data-min-compiler-version="1.1"> 

``` kotlin
import kotlin.coroutines.experimental.*

fun main(args: Array<String>) {
//sampleStart
  val seq = buildSequence {
      for (i in 1..5) {
          // yield a square of i
          yield(i * i)
      }
      // yield a range
      yieldAll(26..28)
  }
  
  // print the sequence
  println(seq.toList())
//sampleEnd
}
```

</div>


Run the code above to see the result. Feel free to edit it and run again!

For more information, please refer to the [coroutine documentation](/docs/reference/coroutines.html) and [tutorial](/docs/tutorials/coroutines-basic-jvm.html).

请注意，协程目前还是一个**实验性的功能**，这意味着 Kotlin 团队不承诺
在最终的 1.1 版本时保持该功能的向后兼容性。


## 其他语言功能

### 类型别名

类型别名允许你为现有类型定义备用名称。
这对于泛型类型（如集合）以及函数类型最有用。
这里有几个例子：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
typealias OscarWinners = Map<String, String>

fun countLaLaLand(oscarWinners: OscarWinners) =
        oscarWinners.count { it.value.contains("La La Land") }

// Note that the type names (initial and the type alias) are interchangeable:
fun checkLaLaLandIsTheBestMovie(oscarWinners: Map<String, String>) =
        oscarWinners["Best picture"] == "La La Land"
//sampleEnd

fun oscarWinners(): OscarWinners {
    return mapOf(
            "Best song" to "City of Stars (La La Land)",
            "Best actress" to "Emma Stone (La La Land)",
            "Best picture" to "Moonlight" /* ... */)
}

fun main(args: Array<String>) {
    val oscarWinners = oscarWinners()

    val laLaLandAwards = countLaLaLand(oscarWinners)
    println("LaLaLandAwards = $laLaLandAwards (in our small example), but actually it's 6.")

    val laLaLandIsTheBestMovie = checkLaLaLandIsTheBestMovie(oscarWinners)
    println("LaLaLandIsTheBestMovie = $laLaLandIsTheBestMovie")
}
```
</div>

更多详细信息请参阅其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/type-aliases.md)。


### 已绑定的可调用引用

现在可以使用 `::` 操作符来获取指向特定对象实例的方法或属性的[成员引用](reflection.html#函数引用)。
以前这只能用 lambda 表达式表示。
这里有一个例子：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
val numberRegex = "\\d+".toRegex()
val numbers = listOf("abc", "123", "456").filter(numberRegex::matches)
//sampleEnd

fun main(args: Array<String>) {
    println("Result is $numbers")
}
```
</div>


更多详细信息请参阅其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/bound-callable-references.md)。


### 密封类和数据类

Kotlin 1.1 删除了一些对 Kotlin 1.0 中已存在的密封类和数据类的限制。
现在你可以在同一个文件中的任何地方定义一个密封类的子类，而不只是以作为密封类嵌套类的方式。
数据类现在可以扩展其他类。
这可以用来友好且清晰地定义一个表达式类的层次结构：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
sealed class Expr

data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()

fun eval(expr: Expr): Double = when (expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
}
val e = eval(Sum(Const(1.0), Const(2.0)))
//sampleEnd

fun main(args: Array<String>) {
    println("e is $e") // 3.0
}
```
</div>

更多详细信息请参阅 the [documentation](sealed-classes.html#密封类的放宽规则（自-11-起）) or 
[密封类](https://github.com/Kotlin/KEEP/blob/master/proposals/sealed-class-inheritance.md) 及
[数据类](https://github.com/Kotlin/KEEP/blob/master/proposals/data-class-inheritance.md)的 KEEP。


### lambda 表达式中的解构

现在可以使用[解构声明](multi-declarations.html)语法来解开传递给 lambda 表达式的参数。
这里有一个例子：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val map = mapOf(1 to "one", 2 to "two")
    // before
    println(map.mapValues { entry ->
        val (key, value) = entry
        "$key -> $value!"
    })
    // now
    println(map.mapValues { (key, value) -> "$key -> $value!" })
//sampleEnd    
}
```
</div>

更多详细信息请参阅 the [documentation](multi-declarations.html#destructuring-in-lambdas-since-11) 及其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/destructuring-in-parameters.md)。


### 下划线用于未使用参数

对于具有多个参数的 lambda 表达式，可以使用 `_` 字符替换不使用的参数的名称：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
    val map = mapOf(1 to "one", 2 to "two")

//sampleStart
    map.forEach { _, value -> println("$value!") }
//sampleEnd    
}
```
</div>

这也适用于[解构声明](multi-declarations.html)：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
data class Result(val value: Any, val status: String)

fun getResult() = Result(42, "ok").also { println("getResult() returns $it") }

fun main(args: Array<String>) {
//sampleStart
    val (_, status) = getResult()
//sampleEnd
    println("status is '$status'")
}
```
</div>

更多详细信息请参阅其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/underscore-for-unused-parameters.md)。


### 数字字面值中的下划线

正如在 Java 8 中一样，Kotlin 现在允许在数字字面值中使用下划线来分隔数字分组：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
val oneMillion = 1_000_000
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
//sampleEnd

fun main(args: Array<String>) {
    println(oneMillion)
    println(hexBytes.toString(16))
    println(bytes.toString(2))
}
```
</div>

更多详细信息请参阅其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/underscores-in-numeric-literals.md)。


### 对于属性的更短语法

对于没有自定义访问器、或者将 getter 定义为表达式主体的属性，现在可以省略属性的类型：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
data class Person(val name: String, val age: Int) {
    val isAdult get() = age >= 20 // Property type inferred to be 'Boolean'
}
//sampleEnd

fun main(args: Array<String>) {
    val akari = Person("Akari", 26)
    println("$akari.isAdult = ${akari.isAdult}")
}
```
</div>

### 内联属性访问器

如果属性没有幕后字段，现在可以使用 `inline` 修饰符来标记该属性访问器。
这些访问器的编译方式与[内联函数](inline-functions.html)相同。

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
public val <T> List<T>.lastIndex: Int
    inline get() = this.size - 1
//sampleEnd

fun main(args: Array<String>) {
    val list = listOf('a', 'b')
    // the getter will be inlined
    println("Last index of $list is ${list.lastIndex}")
}
```
</div>

You can also mark the entire property as `inline` - then the modifier is applied to both accessors.

更多详细信息请参阅 the [documentation](inline-functions.html#inline-properties-since-11) 及其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/inline-properties.md)。


### 局部委托属性

现在可以对局部变量使用[委托属性](delegated-properties.html)语法。
一个可能的用途是定义一个延迟求值的局部变量：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
import java.util.Random

fun needAnswer() = Random().nextBoolean()

fun main(args: Array<String>) {
//sampleStart
    val answer by lazy {
        println("Calculating the answer...")
        42
    }
    if (needAnswer()) {                     // returns the random value
        println("The answer is $answer.")   // answer is calculated at this point
    }
    else {
        println("Sometimes no answer is the answer...")
    }
//sampleEnd
}
```
</div>

更多详细信息请参阅其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/local-delegated-properties.md)。


### 委托属性绑定的拦截

对于[委托属性](delegated-properties.html)，现在可以使用 `provideDelegate` 操作符拦截委托到属性的绑定
。
例如，如果我们想要在绑定之前检查属性名称，我们可以这样写：

``` kotlin
class ResourceLoader<T>(id: ResourceID<T>) {
    operator fun provideDelegate(thisRef: MyUI, property: KProperty<*>): ReadOnlyProperty<MyUI, T> {
        checkProperty(thisRef, property.name)
        …… // 属性创建
    }

    private fun checkProperty(thisRef: MyUI, name: String) { …… }
}

fun <T> bindResource(id: ResourceID<T>): ResourceLoader<T> { …… }

class MyUI {
    val image by bindResource(ResourceID.image_id)
    val text by bindResource(ResourceID.text_id)
}
```

`provideDelegate` 方法在创建 `MyUI` 实例期间将会为每个属性调用，并且可以立即执行
必要的验证。

Read the [documentation](delegated-properties.html#providing-a-delegate-since-11) for more details.


### 泛型枚举值访问

现在可以用泛型的方式来枚举枚举类的值：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
//sampleStart
enum class RGB { RED, GREEN, BLUE }

inline fun <reified T : Enum<T>> printAllValues() {
    print(enumValues<T>().joinToString { it.name })
}
//sampleEnd

fun main(args: Array<String>) {
    printAllValues<RGB>() // 输出 RED, GREEN, BLUE
}
```
</div>

### 对于 DSL 中隐式接收者的作用域控制

`@DslMarker` 注解允许限制来自 DSL 上下文中的外部作用域的接收者的使用。
考虑那个典型的 [HTML 构建器示例](type-safe-builders.html)：

``` kotlin
table {
    tr {
        td { +"Text" }
    }
}
```

在 Kotlin 1.0 中，传递给 `td` 的 lambda 表达式中的代码可以访问三个隐式接收者：传递给 `table`、`tr`
和 `td` 的。 这允许你调用在上下文中没有意义的方法——例如在 `td` 里面调用 `tr`，从而
在 `<td>` 中放置一个 `<tr>` 标签。

在 Kotlin 1.1 中，你可以限制这种情况，以使只有在 `td` 的隐式接收者上定义的方法
会在传给 `td` 的 lambda 表达式中可用。你可以通过定义标记有 `@DslMarker` 元注解的注解
并将其应用于标记类的基类：

更多详细信息请参阅 the [documentation](type-safe-builders.html#scope-control-dslmarker-since-11) 及其 [KEEP](https://github.com/Kotlin/KEEP/blob/master/proposals/scope-control-for-implicit-receivers.md)。


### `rem` 操作符

`mod` 操作符现已弃用，而使用 `rem` 取代。动机参见[这个问题](https://youtrack.jetbrains.com/issue/KT-14650)。

## 标准库

### 字符串到数字的转换

在 String 类中有一些新的扩展，用来将它转换为数字，而不会在无效数字上抛出异常：
`String.toIntOrNull(): Int?`、 `String.toDoubleOrNull(): Double?` 等。

```
val port = System.getenv("PORT")?.toIntOrNull() ?: 80
```

还有整数转换函数，如 `Int.toString()`、 `String.toInt()`、 `String.toIntOrNull()`，
每个都有一个带有 `radix` 参数的重载，它允许指定转换的基数（2 到 36）。

### onEach()

`onEach` 是一个小、但对于集合和序列很有用的扩展函数，它允许对操作链中
的集合/序列的每个元素执行一些操作，可能带有副作用。
对于迭代其行为像 `forEach` 但是也进一步返回可迭代实例。 对于序列它返回一个
包装序列，它在元素迭代时延迟应用给定的动作。

``` kotlin
inputDir.walk()
        .filter { it.isFile && it.name.endsWith(".txt") }
        .onEach { println("Moving $it to $outputDir") }
        .forEach { moveFile(it, File(outputDir, it.toRelativeString(inputDir))) }
```

### also()、takeIf() 和 takeUnless()

这些是适用于任何接收者的三个通用扩展函数。
 
`also` 就像 `apply`：它接受接收者、做一些动作、并返回该接收者。
二者区别是在 `apply` 内部的代码块中接收者是 `this`，
而在 `also` 内部的代码块中是 `it`（并且如果你想的话，你可以给它另一个名字）。
当你不想掩盖来自外部作用域的 `this` 时这很方便：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
class Block {
    lateinit var content: String
}

//sampleStart
fun Block.copy() = Block().also {
    it.content = this.content
}
//sampleEnd

// using 'apply' instead
fun Block.copy1() = Block().apply {
    this.content = this@copy1.content
}

fun main(args: Array<String>) {
    val block = Block().apply { content = "content" }
    val copy = block.copy()
    println("Testing the content was copied:")
    println(block.content == copy.content)
}
```
</div>

`takeIf` 就像单个值的 `filter`。它检查接收者是否满足该谓词，并
在满足时返回该接收者否则不满足时返回 `null`。
结合 elvis-操作符和及早返回，它允许编写如下结构：

``` kotlin
val outDirFile = File(outputDir.path).takeIf { it.exists() } ?: return false
// 对现有的 outDirFile 做些事情
```

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
    val input = "Kotlin"
    val keyword = "in"

//sampleStart
    val index = input.indexOf(keyword).takeIf { it >= 0 } ?: error("keyword not found")
    // 对输入字符串中的关键字索引做些事情，假定它找到
//sampleEnd
    
    println("'$keyword' was found in '$input'")
    println(input)
    println(" ".repeat(index) + "^")
}
```
</div>

`takeUnless` 与 `takeIf` 相同，只是它采用了反向谓词。当它 _不_ 满足谓词时返回接收者，否则返回 `null`。因此，上面的示例之一可以用 `takeUnless` 重写如下：

``` kotlin
val index = input.indexOf(keyword).takeUnless { it < 0 } ?: error("keyword not found")
```

当你有一个可调用的引用而不是 lambda 时，使用也很方便：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
private fun testTakeUnless(string: String) {
//sampleStart
    val result = string.takeUnless(String::isEmpty)
//sampleEnd

    println("string = \"$string\"; result = \"$result\"")
}

fun main(args: Array<String>) {
    testTakeUnless("")
    testTakeUnless("abc")
}
```
</div>

### groupingBy()

此 API 可以用于按照键对集合进行分组，并同时折叠每个组。 例如，它可以用于
计算文本中字符的频率：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
    val words = "one two three four five six seven eight nine ten".split(' ')
//sampleStart
    val frequencies = words.groupingBy { it.first() }.eachCount()
//sampleEnd
    println("Counting first letters: $frequencies.")

    // The alternative way that uses 'groupBy' and 'mapValues' creates an intermediate map, 
    // while 'groupingBy' way counts on the fly.
    val groupBy = words.groupBy { it.first() }.mapValues { (_, list) -> list.size }
    println("Comparing the result with using 'groupBy': ${groupBy == frequencies}.")
}
```
</div>

### Map.toMap() 和 Map.toMutableMap()

这俩函数可以用来简易复制映射：

``` kotlin
class ImmutablePropertyBag(map: Map<String, Any>) {
    private val mapCopy = map.toMap()
}
```

### Map.minus(key)

运算符 `plus` 提供了一种将键值对添加到只读映射中以生成新映射的方法，但是没有一种简单的方法来做相反的操作：从映射中删除一个键采用不那么直接的方式如 Map.filter() 或 Map.filterKeys()。
现在运算符 `minus` 填补了这个空白。有 4 个可用的重载：用于删除单个键、键的集合、键的序列和键的数组。

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val map = mapOf("key" to 42)
    val emptyMap = map - "key"
//sampleEnd
    
    println("map: $map")
    println("emptyMap: $emptyMap")
}
```
</div>

### minOf() 和 maxOf()

这些函数可用于查找两个或三个给定值中的最小和最大值，其中值是原生数字或 `Comparable` 对象。每个函数还有一个重载，它接受一个额外的 `Comparator` 实例，如果你想比较自身不可比的对象的话。

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val list1 = listOf("a", "b")
    val list2 = listOf("x", "y", "z")
    val minSize = minOf(list1.size, list2.size)
    val longestList = maxOf(list1, list2, compareBy { it.size })
//sampleEnd
    
    println("minSize = $minSize")
    println("longestList = $longestList")
}
```
</div>

### 类似数组的列表实例化函数

类似于 `Array` 构造函数，现在有创建 `List` 和 `MutableList` 实例的函数，并通过
调用 lambda 表达式来初始化每个元素：

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val squares = List(10) { index -> index * index }
    val mutable = MutableList(10) { 0 }
//sampleEnd

    println("squares: $squares")
    println("mutable: $mutable")
}
```
</div>

### Map.getValue()

`Map` 上的这个扩展函数返回一个与给定键相对应的现有值，或者抛出一个异常，提示找不到该键。
如果该映射是用 `withDefault` 生成的，这个函数将返回默认值，而不是抛异常。

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {

//sampleStart    
    val map = mapOf("key" to 42)
    // 返回不可空 Int 值 42
    val value: Int = map.getValue("key")

    val mapWithDefault = map.withDefault { k -> k.length }
    // 返回 4
    val value2 = mapWithDefault.getValue("key2")

    // map.getValue("anotherKey") // <- 这将抛出 NoSuchElementException
//sampleEnd
    
    println("value is $value")
    println("value2 is $value2")
}
```
</div>

### 抽象集合

这些抽象类可以在实现 Kotlin 集合类时用作基类。
对于实现只读集合，有 `AbstractCollection`、 `AbstractList`、 `AbstractSet` 和 `AbstractMap`，
而对于可变集合，有 `AbstractMutableCollection`、 `AbstractMutableList`、 `AbstractMutableSet` 和 `AbstractMutableMap`。
在 JVM 上，这些抽象可变集合从 JDK 的抽象集合继承了大部分的功能。

### Array manipulation functions

The standard library now provides a set of functions for element-by-element operations on arrays: comparison
(`contentEquals` and `contentDeepEquals`), hash code calculation (`contentHashCode` and `contentDeepHashCode`),
and conversion to a string (`contentToString` and `contentDeepToString`). They're supported both for the JVM
(where they act as aliases for the corresponding functions in `java.util.Arrays`) and for JS (where the implementation
is provided in the Kotlin standard library).

<div class="sample" markdown="1" data-min-compiler-version="1.1">

``` kotlin
fun main(args: Array<String>) {
//sampleStart
    val array = arrayOf("a", "b", "c")
    println(array.toString())  // JVM implementation: type-and-hash gibberish
    println(array.contentToString())  // nicely formatted as list
//sampleEnd
}
```
</div>

## JVM 后端

### Java 8 字节码支持

Kotlin 现在可以选择生成 Java 8 字节码（命令行选项 `-jvm-target 1.8`或者Ant/Maven/Gradle 中
的相应选项）。目前这并不改变字节码的语义（特别是，接口和 lambda 表达式中的默认方法
的生成与 Kotlin 1.0 中完全一样），但我们计划在以后进一步使用它。


### Java 8 标准库支持

现在有支持在 Java 7 和 8 中新添加的 JDK API 的标准库的独立版本。
如果你需要访问新的 API，请使用 `kotlin-stdlib-jre7` 和 `kotlin-stdlib-jre8` maven artifacts，而不是标准的 `kotlin-stdlib`。
这些构件是在 `kotlin-stdlib` 之上的微小扩展，它们将它作为传递依赖项带到项目中。


### 字节码中的参数名

Kotlin 现在支持在字节码中存储参数名。这可以使用命令行选项 `-java-parameters` 启用。


### 常量内联

编译器现在将 `const val` 属性的值内联到使用它们的位置。


### 可变闭包变量

用于在 lambda 表达式中捕获可变闭包变量的装箱类不再具有 volatile 字段。
此更改提高了性能，但在一些罕见的使用情况下可能导致新的竞争条件。如果受此影响，你需要提供
自己的同步机制来访问变量。


### javax.scripting 支持

Kotlin 现在与[javax.script API](https://docs.oracle.com/javase/8/docs/api/javax/script/package-summary.html)（JSR-223）集成。
The API allows to evaluate snippets of code at runtime:

``` kotlin
val engine = ScriptEngineManager().getEngineByExtension("kts")!!
engine.eval("val x = 3")
println(engine.eval("x + 2"))  // Prints out 5
```

关于使用 API 的示例项目参见[这里](https://github.com/JetBrains/kotlin/tree/master/libraries/examples/kotlin-jsr223-local-example)
。


### kotlin.reflect.full

To [prepare for Java 9 support](https://blog.jetbrains.com/kotlin/2017/01/kotlin-1-1-whats-coming-in-the-standard-library/), the extension functions and properties in the `kotlin-reflect.jar` library have been moved
to the package `kotlin.reflect.full`. The names in the old package (`kotlin.reflect`) are deprecated and will be removed in
Kotlin 1.2. Note that the core reflection interfaces (such as `KClass`) are part of the Kotlin standard library,
not `kotlin-reflect`, and are not affected by the move.


## JavaScript 后端

### 统一的标准库

Kotlin 标准库的大部分目前可以从代码编译成 JavaScript 来使用。
特别是，关键类如集合（`ArrayList`、 `HashMap` 等）、异常（`IllegalArgumentException` 等）以及其他
几个关键类（`StringBuilder`、 `Comparator`）现在都定义在 `kotlin` 包下。在 JVM 平台上，一些名称是相应 JDK 类的
类型别名，而在 JS 平台上，这些类在 Kotlin 标准库中实现。

### 更好的代码生成

JavaScript 后端现在生成更加可静态检查的代码，这对 JS 代码处理工具（如
minifiers、 optimisers、 linters 等）更加友好。

### `external` 修饰符

如果你需要以类型安全的方式从 Kotlin 访问 JavaScript 实现的类，
你可以使用 `external` 修饰符写一个 Kotlin 声明。（在 Kotlin 1.0 中，使用了 `@native` 注解。）
与 JVM 目标平台不同，JS 平台允许对类和属性使用外部修饰符。
例如，可以按以下方式声明 DOM `Node` 类：

``` kotlin
external class Node {
    val firstChild: Node

    fun appendChild(child: Node): Node

    fun removeChild(child: Node): Node

    // 等等
}
```

### 改进的导入处理

现在可以更精确地描述应该从 JavaScript 模块导入的声明。
如果在外部声明上添加 `@JsModule("<module-name>")` 注解，它会在编译期间正确导入
到模块系统（CommonJS或AMD）。例如，使用 CommonJS，该声明会
通过 `require(...)` 函数导入。
此外，如果要将声明作为模块或全局 JavaScript 对象导入，
可以使用 `@JsNonModule` 注解。

例如，以下是将 JQuery 导入 Kotlin 模块的方法：

``` kotlin
external interface JQuery {
    fun toggle(duration: Int = definedExternally): JQuery
    fun click(handler: (Event) -> Unit): JQuery
}

@JsModule("jquery")
@JsNonModule
@JsName("$")
external fun jquery(selector: String): JQuery
```

在这种情况下，JQuery 将作为名为 `jquery` 的模块导入。或者，它可以用作 $-对象，
这取决于Kotlin编译器配置使用哪个模块系统。

你可以在应用程序中使用如下所示的这些声明：

``` kotlin
fun main(args: Array<String>) {
    jquery(".toggle-button").click {
        jquery(".toggle-panel").toggle(300)
    }
}
```
