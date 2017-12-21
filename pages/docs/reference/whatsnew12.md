---
type: doc
layout: reference
title: "Kotlin 1.2 的新特性"
---

# Kotlin 1.2 的新特性

## 目录

* [多平台项目](#multiplatform-projects-experimental)
* [其它语言特性](#other-language-features)
* [标准库](#standard-library)
* [JVM backend](#jvm-backend)
* [JavaScript backend](#javascript-backend)

## 多平台项目 (实验性的)

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

When a variable is assigned from a safe call expression and checked for null, the smart cast is now applied to 
the safe call receiver as well:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun countFirst(s: Any): Int {
    //sampleStart
    val firstChar = (s as? CharSequence)?.firstOrNull()
    if (firstChar != null)
       return s.count { it == firstChar } // s: Any is smart cast to CharSequence
    
    val firstItem = (s as? Iterable<*>)?.firstOrNull()
    if (firstItem != null)
       return s.count { it == firstItem } // s: Any is smart cast to Iterable<*>
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

Also, smart casts in a lambda are now allowed for local variables that are only modified before the lambda:

<div class="sample" markdown="1" data-min-compiler-version="1.2">

```kotlin
fun main(args: Array<String>) {
    val flag = args.size == 0
    
    //sampleStart
    var x: String? = null
    if (flag) x = "Yahoo!"

    run {
        if (x != null) {
            println(x.length) // x is smart cast to String
        }
    }
    //sampleEnd
}
```
</div>

### Support for  ::foo as a shorthand for this::foo

A bound callable reference to a member of `this` can now be written without explicit receiver, `::foo` instead 
of `this::foo`. This also makes callable references more convenient to use in lambdas where you refer to a member 
of the outer receiver.

### Breaking change: sound smart casts after try blocks

Earlier, Kotlin used assignments made inside a `try` block for smart casts after the block, which could break type- and null-safety 
and lead to runtime failures. This release fixes this issue, making the smart casts more strict, but breaking some code
that relied on such smart casts.

To switch to the old smart casts behavior, pass the fallback flag `-Xlegacy-smart-cast-after-try` as the compiler 
argument. It will become deprecated in Kotlin 1.3.

### Deprecation: data classes overriding copy

When a data class derived from a type that already had the `copy` function with the same signature, the `copy` 
implementation generated for the data class used the defaults from the supertype, leading to counter-intuitive behavior, 
or failed at runtime if there were no default parameters in the supertype. 

Inheritance that leads to a `copy` conflict has become deprecated with a warning in Kotlin 1.2 
and will be an error in Kotlin 1.3.

### Deprecation: nested types in enum entries

Inside enum entries, defining a nested type that is not an `inner class` has been deprecated due to issues in the 
initialization logic. This causes a warning in Kotlin 1.2 and will become an error in Kotlin 1.3.

### Deprecation: single named argument for vararg

For consistency with array literals in annotations, passing a single item for a vararg parameter in the named 
form (`foo(items = i)`) has been deprecated. Please use the spread operator with the corresponding 
array factory functions:

```kotlin
foo(items = *intArrayOf(1))
```

There is an optimization that removes redundant arrays creation in such cases, which prevents performance degradation. 
The single-argument form produces warnings in Kotlin 1.2 and is to be dropped in Kotlin 1.3.

### Deprecation: inner classes of generic classes extending Throwable

Inner classes of generic types that inherit from `Throwable` could violate type-safety in a throw-catch scenario and 
thus have been deprecated, with a warning in Kotlin 1.2 and an error in Kotlin 1.3.

### Deprecation: mutating backing field of a read-only property

Mutating the backing field of a read-only property by assigning `field = ...` in the custom getter has been 
deprecated, with a warning in Kotlin 1.2 and an error in Kotlin 1.3.

## 标准库

### Kotlin standard library artifacts and split packages

The Kotlin standard library is now fully compatible with the Java 9 module system, which forbids split packages 
(multiple jar files declaring classes in the same package). In order to support that, new artifacts `kotlin-stdlib-jdk7` 
and `kotlin-stdlib-jdk8` are introduced, which replace the old `kotlin-stdlib-jre7` and `kotlin-stdlib-jre8`.
 
The declarations in the new artifacts are visible under the same package names from the Kotlin point of view, but have 
different package names for Java. Therefore, switching to the new artifacts will not require any changes to 
your source code.

Another change made to ensure compatibility with the new module system is removing the deprecated 
declarations in the `kotlin.reflect` package from the `kotlin-reflect` library. If you were using them, you need 
to switch to using the declarations in the `kotlin.reflect.full` package, which is supported since Kotlin 1.1.

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

### Math operations in kotlin-stdlib

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
