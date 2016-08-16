---
type: doc
layout: reference
category: "Syntax"
title: "范围"
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
