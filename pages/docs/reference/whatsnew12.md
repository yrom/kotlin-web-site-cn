---
type: doc
layout: reference
title: "Kotlin 1.2 的新特性"
---

# Kotlin 1.2 的新特性

## 目录

* [多平台项目](#多平台项目实验性的)
* [其它语言特性](#其它语言特性)
* [标准库](#标准库)
* [JVM backend](#jvm-backend)
* [JavaScript backend](#javascript-backend)

## 多平台项目（实验性的）

多平台项目是 Kotlin 1.2 的一个新的**试验性**功能，它允许你在不同的 Kotlin 所支持的目标平台 —— JVM，JavaScript 以及(未来的)Native —— 之间共用代码。在多平台项目中，存在三种类型的模块：

* *通用* 模块，包含了平台无关的代码，以及声明一些跟平台相关的未实现的API。
* *平台* 模块，包含了特定平台对于通用模块里平台相关性API的实现，以及其他平台相关的代码。
* 面向特定平台的普通模块，可以作为平台模块的依赖，也可以依赖于平台模块。

将一个多平台项目编译到指定平台时，会同时生成通用部分和平台指定部分的代码。

多平台项目所支持的一个关键特性是，使得通用代码在平台指定部分的依赖关系可通过**预期(expect)和实际(actual)的声明**表达出来。预期(expect)声明用于指定API（类、接口、注解、顶级声明等），实际(actual)声明用于指定API的平台相关实现或者用`typealias`引用外部库的现有实现。举个例子：

在通用代码中:

```kotlin
// 预期平台指定的API:
expect fun hello(world: String): String

fun greet() {
    // 使用预期 API:
    val greeting = hello("multi-platform world")
    println(greeting)
}

expect class URL(spec: String) {
    open fun getHost(): String
    open fun getPath(): String
}
```

在 JVM 平台代码中:

```kotlin
actual fun hello(world: String): String =
    "Hello, $world, on the JVM platform!"

// 使用现有平台特定实现:
actual typealias URL = java.net.URL
```

详细内容及构建多平台项目的步骤可参见其[文档](http://kotlinlang.org/docs/reference/multiplatform.html)。

## 其它语言特性

### 注解中的数组字面值

从 Kotlin 1.2 开始，注解的数组参数可以用创建数组字面值语法（即用中括号）来取代函数`arrayOf`:

```kotlin
@CacheConfig(cacheNames = ["books", "default"])
public class BookRepositoryImpl {
    // ...
}
```

数组字面值语法仅限用于注解的数组参数。

### 延迟初始化顶级属性和局部变量

修饰符`lateinit`现在可以用于顶级属性和局部变量了。例如，当一个lambda 作为构造函数参数传递给一个对象时，后者可以用于引用一个不得不稍后定义的对象：

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
class Node<T>(val value: T, val next: () -> Node<T>)

fun main(args: Array<String>) {
    //sampleStart
    // 3个node循环引用:
    lateinit var third: Node<Int>
    
    val second = Node(2, next = { third })
    val first = Node(1, next = { second })
    
    third = Node(3, next = { first })
    //sampleEnd
    
    val nodes = generateSequence(first) { it.next() }
    println("Values in the cycle: ${nodes.take(7).joinToString { it.value.toString() }}, ...")
}
```
</div>

### 检查lateinit变量是否被初始化

现在可以用`isInitialized`来检查一个lateinit的变量是否已被初始化：

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
class Foo {
    lateinit var lateinitVar: String
    
    fun initializationLogic() {
        //sampleStart
        println("isInitialized before assignment: " + this::lateinitVar.isInitialized)
        lateinitVar = "value"
        println("isInitialized after assignment: " + this::lateinitVar.isInitialized)    
        //sampleEnd
    }
}

fun main(args: Array<String>) {
	Foo().initializationLogic()
}
```
</div>

### 有默认参数值的内联函数

现在，内联函数的参数也可以有默认值了:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
//sampleStart
inline fun <E> Iterable<E>.strings(transform: (E) -> String = { it.toString() }) = 
    map { transform(it) }

val defaultStrings = listOf(1, 2, 3).strings()
val customStrings = listOf(1, 2, 3).strings { "($it)" } 
//sampleEnd

fun main(args: Array<String>) {
    println("defaultStrings = $defaultStrings")
    println("customStrings = $customStrings")
}
```
</div>

### 显式转换的信息将用于类型推导

Kotlin 的编译器现在可以将从显式类型转换得到信息用于类型推导。假设现在正调用一个返回类型参数为`T`的泛型方法，你将返回值强转为指定类型`Foo`，
编译器这时就会知道这里的 `T` 需要被绑定为类型 `Foo`。

这对于Android 的开发者来说尤其重要，因为编译器现在能正确的分析 Android API 级别 26 中的泛型化的 `findViewById`：

```kotlin
val button = findViewById(R.id.button) as Button
```

### 优化智能转换

当变量被一个安全调用（safe call）表达式赋值并且做了null检查，智能转换现在同样会被应用到安全调用的接收者:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun countFirst(s: Any): Int {
    //sampleStart
    val firstChar = (s as? CharSequence)?.firstOrNull()
    if (firstChar != null)
       return s.count { it == firstChar } // s: Any 被智能转换成 CharSequence
    
    val firstItem = (s as? Iterable<*>)?.firstOrNull()
    if (firstItem != null)
       return s.count { it == firstItem } // s: Any 被智能转换成 Iterable<*>
    //sampleEnd
    
    return -1
}

fun main(args: Array<String>) {
    val string = "abacaba"
    val countInString = countFirst(string)
    println("called on \"$string\": $countInString")
    
    val list = listOf(1, 2, 3, 1, 2)
    val countInList = countFirst(list)
    println("called on $list: $countInList")
}
```
</div>

另外，智能转换现在被允许在 lambda 中应用到那些仅在 lambda 之前被修改的局部变量：:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun main(args: Array<String>) {
    val flag = args.size == 0
    
    //sampleStart
    var x: String? = null
    if (flag) x = "Yahoo!"

    run {
        if (x != null) {
            println(x.length) // x 被智能转换为 String
        }
    }
    //sampleEnd
}
```
</div>

### 支持将 this::foo 简写为 ::foo 

一个绑定到`this`成员的可调用引用(callable reference)，现在可以不写明接收者，如用`::foo`替代`this::foo`。
这也使得可调用引用可以更方便的在lambda中引用外部接收者的成员。

### 破坏性变更: sound smart casts after try blocks

早前，Kotlin 使用`try`代码块中的赋值语句作为块后的智能转换依据，这其实会破坏类型和null的安全性进而导致运行时错误。
这个版本修复了该问题，使智能转换更加严格，但会破坏一些依赖于这个转换规则的代码。

要切换回旧的智能转换行为，可以使用编译器参数标识 `-Xlegacy-smart-cast-after-try`。该标识将在 Kotlin 1.3 版本中被废弃。

### 废弃: 数据类（data classes）的重载拷贝（copy）

当从一个具有相同签名的`copy`函数的类型派生出一个数据类时，生成的`copy`函数实现会使用父类的默认值，这是反直觉的行为，
甚至在运行时会执行失败，如果父类并没有默认参数的话。

引起`copy`冲突的这种继承行为已经被废弃了，Kotlin 1.2 版本中会体现为警告，而在1.3中将是错误。

### 废弃: 枚举（enum）条目中的嵌套类型

在枚举条目中定义一个不是`内部类`的嵌套类型，因为初始化逻辑有问题而被废弃了。在 Kotlin 1.2 版本中会引发警告，在1.3中将是错误。

### 废弃: 为可变参数（vararg）的单个参数命名

为了与注解中的数组字面值保持一致性，使用命名的形式(`foo(items = i)`)给可变参数传递单个条目已被废弃。
请使用具有相应数组工厂函数的扩展操作符：

```kotlin
foo(items = *intArrayOf(1))
```

在这种情况下，有一种优化可以消除重复数组的创建，以避免性能下降。命名单参数在 Kotlin 1.2 版本中会引发警告，在1.3中将被丢弃。 

### 废弃: 继承Throwable的泛型内部类

从`Throwable`继承的泛型内部类可能会违反类型安全性，因此已被弃用。在Kotlin 1.2中为警告，在 1.3 中为错误。

### 废弃: 修改只读属性背后的字段

在自定义的getter中用赋值语句`field = ...`来修改只读属性背后的字段，已经被废弃了。在Kotlin 1.2中为警告，在 1.3 中为错误。

## 标准库

### Kotlin 标准库构件和拆分包

Kotlin 的标准库现在完全兼容 Java 9 的模块系统，它禁止拆分包（多个 jar 文件声明同个包名的类）。为此，引入了新的构件
`kotlin-stdlib-jdk7` 和 `kotlin-stdlib-jdk8`，用来替换旧的 `kotlin-stdlib-jre7` 和 `kotlin-stdlib-jre8`。

新构件中的声明在 Kotlin 中为同个包名，而Java中则为不同包名。因此，切换到新构件并不要求对源码作任何改动。

另一个为了兼容新模块系统的改动是从 `kotlin-reflect` 库中移除了 `kotlin.reflect` 包中已被废弃了的声明。
如果你还在用它们，需要切换为 `kotlin.reflect.full` 包的声明，该包从 Kotlin 1.1 版本起就已支持。

### windowed, chunked, zipWithNext

New extensions for `Iterable<T>`, `Sequence<T>`, and `CharSequence` cover such use cases as buffering or 
batch processing (`chunked`), sliding window and computing sliding average (`windowed`) , and processing pairs 
of subsequent items (`zipWithNext`):

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun main(args: Array<String>) {
    //sampleStart
    val items = (1..9).map { it * it }

    val chunkedIntoLists = items.chunked(4)
    val points3d = items.chunked(3) { (x, y, z) -> Triple(x, y, z) }
    val windowed = items.windowed(4)
    val slidingAverage = items.windowed(4) { it.average() }
    val pairwiseDifferences = items.zipWithNext { a, b -> b - a }
    //sampleEnd
    
    println("items: $items\n")
    
    println("chunked into lists: $chunkedIntoLists")
    println("3D points: $points3d")
    println("windowed by 4: $windowed")
    println("sliding average by 4: $slidingAverage")
    println("pairwise differences: $pairwiseDifferences")
}
```
</div>

### fill, replaceAll, shuffle/shuffled

A set of extension functions was added for manipulating lists: `fill`, `replaceAll` and `shuffle` for `MutableList`, 
and `shuffled` for read-only `List`:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun main(args: Array<String>) {
    //sampleStart
    val items = (1..5).toMutableList()
    
    items.shuffle()
    println("Shuffled items: $items")
    
    items.replaceAll { it * 2 }
    println("Items doubled: $items")
    
    items.fill(5)
    println("Items filled with 5: $items")
    //sampleEnd
}
```
</div>

### kotlin-stdlib 中的数学运算

Satisfying the longstanding request, Kotlin 1.2 adds the `kotlin.math` API for math operations that is common 
for JVM and JS and contains the following:

* Constants: `PI` and `E`;
* Trigonometric: `cos`, `sin`, `tan` and inverse of them: `acos`, `asin`, `atan`, `atan2`;
* Hyperbolic: `cosh`, `sinh`, `tanh` and their inverse: `acosh`, `asinh`, `atanh`
* Exponentation: `pow` (an extension function), `sqrt`, `hypot`, `exp`, `expm1`;
* Logarithms: `log`, `log2`, `log10`, `ln`, `ln1p`;
* Rounding:
    * `ceil`, `floor`, `truncate`, `round` (half to even) functions;
    * `roundToInt`, `roundToLong` (half to integer) extension functions;
* Sign and absolute value:
    * `abs` and `sign` functions;
    * `absoluteValue` and `sign` extension properties;
    * `withSign` extension function;
* `max` and `min` of two values;
* Binary representation:
    * `ulp` extension property;
    * `nextUp`, `nextDown`, `nextTowards` extension functions;
    * `toBits`, `toRawBits`, `Double.fromBits` (these are in the `kotlin` package).

The same set of functions (but without constants) is also available for `Float` arguments.

### Operators and conversions for BigInteger and BigDecimal

Kotlin 1.2 introduces a set of functions for operating with `BigInteger` and `BigDecimal` and creating them 
from other numeric types. These are:

* `toBigInteger` for `Int` and `Long`;
* `toBigDecimal` for `Int`, `Long`, `Float`, `Double`, and `BigInteger`;
* Arithmetic and bitwise operator functions:
    * Binary operators  `+`, `-`, `*`, `/`, `%` and infix functions `and`, `or`, `xor`, `shl`, `shr`;
    * Unary operators `-`, `++`, `--`, and a function `inv`.

### Floating point to bits conversions

New functions were added for converting `Double` and `Float` to and from their bit representations:

* `toBits` and `toRawBits` returning `Long` for `Double` and `Int` for `Float`;
* `Double.fromBits` and `Float.fromBits` for creating floating point numbers from the bit representation.

### Regex is now serializable

The `kotlin.text.Regex` class has become `Serializable` and can now be used in serializable hierarchies.

### Closeable.use calls Throwable.addSuppressed if available

The `Closeable.use` function calls `Throwable.addSuppressed` when an exception is thrown during closing the resource 
after some other exception. 

To enable this behavior you need to have `kotlin-stdlib-jdk7` in your dependencies.

## JVM Backend

### Constructor calls normalization

Ever since version 1.0, Kotlin supported expressions with complex control flow, such as try-catch expressions and 
inline function calls. Such code is valid according to the Java Virtual Machine specification. Unfortunately, some 
bytecode processing tools do not handle such code quite well when such expressions are present in the arguments 
of constructor calls.

To mitigate this problem for the users of such bytecode processing tools, we’ve added a command-line 
option (`-Xnormalize-constructor-calls=MODE`) that tells the compiler to generate more Java-like bytecode for such 
constructs. Here `MODE` is one of:

* `disable` (default) – generate bytecode in the same way as in Kotlin 1.0 and 1.1;
* `enable` – generate Java-like bytecode for constructor calls. This can change the order in which the classes are 
 loaded and initialized;
* `preserve-class-initialization` – generate Java-like bytecode for constructor calls, ensuring that the class 
initialization order is preserved. This can affect overall performance of your application; use it only if you have 
some complex state shared between multiple classes and updated on class initialization.

The “manual” workaround is to store the values of sub-expressions with control flow in variables, instead of 
evaluating them directly inside the call arguments. It’s similar to `-Xnormalize-constructor-calls=enable`.

### Java-default method calls 

Before Kotlin 1.2, interface members overriding Java-default methods while targeting JVM 1.6 produced a warning on 
super calls: `Super calls to Java default methods are deprecated in JVM target 1.6. Recompile with '-jvm-target 1.8'`. 
In Kotlin 1.2, there's an **error** instead, thus requiring any such code to be compiled with JVM target 1.8.

### Breaking change: consistent behavior of x.equals(null) for platform types

Calling `x.equals(null)` on a platform type that is mapped to a Java primitive 
(`Int!`, `Boolean!`, `Short`!, `Long!`, `Float!`, `Double!`, `Char!`) incorrectly returned `true` when `x` was null. 
Starting with Kotlin 1.2, calling `x.equals(...)` on a null value of a platform type **throws an NPE** 
(but `x == ...` does not).

To return to the pre-1.2 behavior, pass the flag `-Xno-exception-on-explicit-equals-for-boxed-null` to the compiler.

### Breaking change: fix for platform null escaping through an inlined extension receiver

Inline extension functions that were called on a null value of a platform type did not check the receiver for null and 
would thus allow null to escape into the other code. Kotlin 1.2 forces this check at the call sites, throwing an exception
if the receiver is null.

To switch to the old behavior, pass the fallback flag `-Xno-receiver-assertions` to the compiler.

## JavaScript Backend

### TypedArrays support enabled by default

The JS typed arrays support that translates Kotlin primitive arrays, such as `IntArray`, `DoubleArray`, 
into [JavaScript typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays), that was 
previously an opt-in feature, has been enabled by default.

## Tools

### Warnings as errors

The compiler now provides an option to treat all warnings as errors. Use `-Werror` on the command line, or the 
following Gradle snippet:

```groovy
compileKotlin {
    kotlinOptions.allWarningsAsErrors = true
}
```
