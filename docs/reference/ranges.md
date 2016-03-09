---
type: doc
layout: reference
category: "Syntax"
title: "Ranges"
---

# 范围

范围表达式是由“rangeTo”函数组成的，操作符的形式是`..`由*in*{: .keyword }和*!in*{: .keyword }补充。
范围被定义为任何可比类型,但是用于原生类型有更优化的实现。下面是使用范围的例子

``` kotlin
if (i in 1..10) { // equivalent of 1 <= i && i <= 10
  println(i)
}
```

整型范围（`IntRange`, `LongRange`, `CharRange`）有一个额外的功能:他们可以遍历。
编译者需要关心的转换是简单模拟Java的索引*for*{: .keyword }循环,不用担心越界。例如：

``` kotlin
for (i in 1..4) print(i) // prints "1234"

for (i in 4..1) print(i) // prints nothing
```

你想要遍历数字颠倒顺序吗?这很简单。您可以使用标准库里面的`downTo()`函数

``` kotlin
for (i in 4 downTo 1) print(i) // prints "4321"
```

是否可以任意进行数量的迭代,而不必每次的变化都是1呢?当然, `step()`函数可以实现

``` kotlin
for (i in 1..4 step 2) print(i) // prints "13"

for (i in 4 downTo 1 step 2) print(i) // prints "42"
```


## 它是如何工作的

Ranges implement a common interface in the library: `ClosedRange<T>`.

`ClosedRange<T>` 在数学意义上表示一个间隔,是对比较类型的定义。
它有两个端点:‘start’和‘endInclusive’,这是包含在范围内。
主要的操作是`contains`,通常用*in*{: .keyword } /*!in* {: .keyword }操作符内。

Integral type progressions (`IntProgression`, `LongProgression`, `CharProgression`) denote an arithmetic progression.
Progressions are defined by the `first` element, the `last` element and a non-zero `increment`.
The first element is `first`, subsequent elements are the previous element plus `increment`. The `last` element is always hit by iteration unless the progression is empty.

A progression is a subtype of `Iterable<N>`, where `N` is `Int`, `Long` or `Char` respectively, so it can be used in *for*{: .keyword }-loops and functions like `map`, `filter`, etc.
迭代`Progression`与Java/JavaScript的基于索引的*for*{: .keyword }循环等价:

``` java
for (int i = first; i != last; i += increment) {
  // ...
}
```

对于整型, `..`操作符创建一个对象既实现了`ClosedRange`也实现了`Progression`。
For example, `IntRange` implements `ClosedRange<Int>` and extends `IntProgression`, thus all operations defined for `IntProgression` are available for `IntRange` as well.
`downTo()`和 `step()`函数的结果一直是`Progression`。

Progressions are constructed with the `fromClosedRange` function defined in their companion objects:

``` kotlin
  IntProgression.fromClosedRange(start, end, increment)
```

The `last` element of the progression is calculated to find maximum value not greater than the `end` value for positive `increment` or minimum value not less than the `end` value for negative `increment` such that `(last - first) % increment == 0`.



> ~~###常见的接口定义~~
> 
> 有两种基本接口:`Range`和`Progression`。
> 
> `Range` 接口定义了一个范围或一个数学意义上的区间。
> 它有两个端点,`start` 和`end`,并且`contains()`函数检查是否包含一个给定的数字范围
> (也可以作为*in*{: .keyword } /*!in*{: .keyword }操作符)。
> “开始”和“结束”是包含在范围内。如果`start`= =`end`,范围包含一个确定的元素。
> 如果 `start` > `end`,范围是空的.
> 
> ``` kotlin
> interface Range<T : Comparable<T>> {
>   val start: T
>   val end: T
>   fun contains(element: T): Boolean
> }
> ```
> 
> `Progression`定义了一种等差算法。
> 它有 `start`(进程中的第一个元素), `end`(被包含的最后一个元素)
> 和`increment` (每个进程元素和以前的区别,非零)。
> 但它的主要特征是,可以遍历过程,所以这是`Iterable`的子类。
> `end`最后一个元素不是必须的，如 `start < end && increment < 0` or `start > end && increment > 0`.
> 
> ``` kotlin
> interface Progression<N : Number> : Iterable<N> {
>   val start: N
>   val end: N
>   val increment: Number // not N, because for Char we'll want it to be negative sometimes
>   // fun iterator(): Iterator<N> is defined in Iterable interface
> }
> ```
> 
> 迭代'Progression'相当于一个索引*for* {:.Java关键字}循环:
> 
> ``` java
> // if increment > 0
> for (int i = start; i <= end; i += increment) {
>   // ...
> }
> 
> // if increment < 0
> for (int i = start; i >= end; i += increment) {
>   // ...
> }
> ```
> 
> 
> ### 类的实现
> 
> 为了避免不必要的重复,让我们只考虑一个数字类型,`Int`。
> 对于其他类型的数量实现是一样的。
> 注意,可以使用这些类的构造函数创建实例,
> 而更方便使用的`rangeTo()`(这个名字,或作为`..`操作符), `downTo()`, `reversed()`和`step()`等实用的函数,以后介绍。
> 
> `IntProgression` 类很简单快捷:
> 
> ``` kotlin
> class IntProgression(override val start: Int, override val end: Int, override val increment: Int): Progression<Int> {
>   override fun iterator(): Iterator<Int> = IntProgressionIteratorImpl(start, end, increment) // implementation of iterator is obvious
> }
> ```
> 
> `IntRange` IntRange是有点复杂:它的实现类是 `Progression<Int>`和`Range<Int>`,因为它是自然的遍历的(默认增量值为1整数和浮点类型):
> 
> ``` kotlin
> class IntRange(override val start: Int, override val end: Int): Range<Int>, Progression<Int> {
>   override val increment: Int
>     get() = 1
>   override fun contains(element: Int): Boolean = start <= element && element <= end
>   override fun iterator(): Iterator<Int> = IntProgressionIteratorImpl(start, end, increment)
> }
> ```
> 
> `ComparableRange` 也很简单(请记住,比较转换是`compareTo()`):
> 
> ``` kotlin
> class ComparableRange<T : Comparable<T>>(override val start: T, override val end: T): Range<T> {
>   override fun contains(element: T): Boolean = start <= element && element <= end
> }
> ```
> 
## 一些实用函数

### `rangeTo()`

整型得`rangeTo()`运算符只要简单地调用构造函数`*Range`类,例如:

``` kotlin
class Int {
  //...
  operator fun rangeTo(other: Long): LongRange = LongRange(this, other)
  //...
  operator fun rangeTo(other: Int): IntRange = IntRange(this, other)
  //...
}
```

Floating point numbers (`Double`, `Float`) do not define their `rangeTo` operator, and the one provided by the standard library for generic `Comparable` types is used instead:

``` kotlin
  public operator fun <T: Comparable<T>> T.rangeTo(that: T): ClosedRange<T>
```

The range returned by this function cannot be used for iteration.

### `downTo()`

`downTo()`的扩展函数可以为任何数字整型对定义,这里有两个例子:

``` kotlin
fun Long.downTo(other: Int): LongProgression {
  return LongProgression.fromClosedRange(this, other, -1.0)
}

fun Byte.downTo(other: Int): IntProgression {
  return IntProgression.fromClosedRange(this, other, -1)
}
```

### `reversed()`

定义`reversed()`扩展函数是为了每个 `*Progression` 类定义的,它们返回反向的级数。

``` kotlin
fun IntProgression.reversed(): IntProgression {
  return IntProgression.fromClosedRange(last, first, -increment)
}
```

### `step()`

`step()`扩展函数是为每个 `*Progression` 类定义的,
他们返回级数与都修改了`step`值(函数参数)。
注意,step值必需总是正的，因此这个函数从不改变的迭代方向。

``` kotlin
fun IntProgression.step(step: Int): IntProgression {
  if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
  return IntProgression.fromClosedRange(first, last, if (increment > 0) step else -step)
}

fun CharProgression.step(step: Int): CharProgression {
  if (step <= 0) throw IllegalArgumentException("Step must be positive, was: $step")
  return CharProgression.fromClosedRange(first, last, step)
}
```

Note that the `last` value of the returned progression may become different from the `last` value of the original progression in order to preserve the invariant `(last - first) % increment == 0`. Here is an example:

``` kotlin
  (1..12 step 2).last == 11  // progression with values [1, 3, 5, 7, 9, 11]
  (1..12 step 3).last == 10  // progression with values [1, 4, 7, 10]
  (1..12 step 4).last == 9   // progression with values [1, 5, 9]
```


