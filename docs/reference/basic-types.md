---
type: doc
layout: reference
category: "Syntax"
title: "Basic Types"
---

# 基本类型

在Kotlin中,所有东西都是对象，所以我们可以调用成员函数和属性的任何变量对象。有些类型是内置的,他们的实现被优化过, 但是用户看起来他们就像普通的类. 本节我们会描述这些类型: numbers, characters, booleans 和 arrays.

## Numbers

Kotlin处理numbers和Java很接近,但是并不完全相同. 例如, 对于numbers没有隐式扩大转换,在一些情况下文字的使用有所不同.

对于numbers Kotlin提供了如下的内置类型 (与Java很相近):

| Type	 | Bitwidth |
|--------|----------|
| Double | 64       |
| Float	 | 32       |
| Long	 | 64       |
| Int	   | 32       |
| Short	 | 16       |
| Byte	 | 8        |

注意 characters 不是 numbers 在Kotlin中。

### Literal Constants

下面是一些常量的写法:

* 十进制: `123`
  * Longs类型用大写 `L` 标记: `123L`
* 十六进制: `0x0F`
* 二进制: `0b00001011`

注意: 不支持8进制

Kotlin 同样支持浮点数的常规表示方法:
 
* Doubles by default: `123.5`, `123.5e10`
* Floats用 `f` 或者 `F` 标记: `123.5f` 

### 表示

在Java平台数字是物理存储为JVM的原始类型,除非我们需要一个可空的引用（例如int？）或泛型. 后者情况下数字被装箱（指的是赋值的时候把实例复制了一下，不是相同实例）。

装箱数字不会保存它的实例:

``` kotlin
val a: Int = 10000
print(a identityEquals a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA identityEquals anotherBoxedA) // !!!Prints 'false'!!!
```
另一方面它们值相等:

``` kotlin
val a: Int = 10000
print(a == a) // Prints 'true'
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // Prints 'true'
```

### 显示转换

由于不同的表示小的类型并不是大类型的子类型。
如果它们是的话，在下面的排序中就会有麻烦：

``` kotlin
// Hypothetical code, does not actually compile:
val a: Int? = 1 // A boxed Int (java.lang.Integer)
val b: Long? = a // implicit conversion yields a boxed Long (java.lang.Long)
print(a == b) // Surprise! This prints "false" as Long's equals() check for other part to be Long as well
```

假设这样是可以的，这里我们就悄无声息的丢掉了一些数据.

因此较小的类型不能隐式转换为较大的类型。
因此我们不能声明一个 `Byte` 类型给一个 `Int` 变量，在不进行显示转换的情况下。

``` kotlin
val b: Byte = 1 // OK, literals are checked statically
val i: Int = b // ERROR
```

我们可以显示转换的去扩大类型

``` kotlin
val i: Int = b.toInt() // OK: explicitly widened
```

每个number类型支持如下的转换:

* `toByte(): Byte`
* `toShort(): Short`
* `toInt(): Int`
* `toLong(): Long`
* `toFloat(): Float`
* `toDouble(): Double`
* `toChar(): Char`

隐式转换是很少被注意的，因为我们使用的类型是从上下文推断的和算数运算符重载的转化，如：

``` kotlin
val l = 1.toLong() + 3 // Long + Int => Long
```

### 运算符

Kotlin支持标准的算数操作符，并在相应的类上定义为成员函数（但编译器会针对运算进行优化，将函数调用优化成直接的算数操作）。
查看 [Operator overloading](operator-overloading.html).

As of bitwise operations, there're no special characters for them, but just named functions that can be called in infix form, for example:

``` kotlin
val x = (1 shl 2) and 0x000FF000
```

这是完整的位运算操作 (只能对 `Int` 或者 `Long` 使用):

* `shl(bits)` – signed shift left (Java's `<<`)
* `shr(bits)` – signed shift right (Java's `>>`)
* `ushr(bits)` – unsigned shift right (Java's `>>>`)
* `and(bits)` – bitwise and
* `or(bits)` – bitwise or
* `xor(bits)` – bitwise xor
* `inv()` – bitwise inversion

## Characters

Characters 用 `Char`来表示. 像对待numbers那样就行

``` kotlin
fun check(c: Char) {
  if (c == 1) { // ERROR: incompatible types
    // ...
  }
}
```

用单引号表示一个Character，例如: `'1'`, `'\n'`, `'\uFF00'`.
我们可以显示的把Character转换为`Int`

``` kotlin
fun decimalDigitValue(c: Char): Int {
  if (c !in '0'..'9')
    throw IllegalArgumentException("Out of range")
  return c.toInt() - '0'.toInt() // Explicit conversions to numbers
}
```

像numbers, characters是被装箱当使用一个可空的引用.这样实例不会被保存。

## Booleans

类型`Boolean`有两个值: *true*{: .keyword } 和 *false*{: .keyword }.

Booleans使用nullable时候Boolean也会被装箱.

内置对Booelan的操作

* `||` – 短路或
* `&&` – 短路与

## 数组

Arrays in Kotlin are represented by the `Array` class, that has `get` and `set` functions (that turn into `[]` by operator overloading conventions), and `size`, along with a few other useful member functions:

``` kotlin
class Array<T> private () {
  fun size(): Int
  fun get(index: Int): T
  fun set(index: Int, value: T): Unit

  fun iterator(): Iterator<T>
  // ...
}
```

To create an array, we can use a library function `array()` and pass the item values to it, so that `array(1, 2, 3)` creates an array [1, 2, 3].
Alternatively, the `arrayOfNulls()` library function can be used to create an array of a given size filled with null elements.

Another option is to use a factory function that takes the array size and the function that can return the initial value
of each array element given its index:

``` kotlin
// Creates an Array<String> with values ["0", "1", "4", "9", "16"]
val asc = Array(5, {i -> (i * i).toString()})
```

As we said above, the `[]` operation stands for calls to member functions `get()` and `set()`.

Note: unlike Java, arrays in Kotlin are invariant. This means that Kotlin does not let us assign an `Array<String>`
to an `Array<Any>`, which prevents a possible runtime failure (but you can use `Array<out Any>`, 
see [Type Projections](generics.html#type-projections)).

Kotlin also has specialized classes to represent arrays of primitive types without boxing overhead: ByteArray,
ShortArray, IntArray and so on. These classes have no inheritance relation to the `Array` class, but they
have the same set of methods and properties. Each of them also has a corresponding factory function:

``` kotlin
val x: IntArray = intArray(1, 2, 3)
x[0] = x[1] + x[2]
```

## Strings

Strings are represented by the type `String`. Strings are immutable.
Elements of a string are characters that can be accessed by the indexing operation: `s[i]`.
A string can be iterated over with a *for*{: .keyword }-loop:

``` kotlin
for (c in str) {
  println(c)
}
```

### String Literals

Kotlin has two types of string literals: escaped strings that may have escaped characters in them and raw strings that can contain newlines and arbitrary text. An escaped string is very much like a Java string:

``` kotlin
val s = "Hello, world!\n"
```

Escaping is done in the conventional way, with a backslash.

A raw string is delimited by a triple quote (`"""`), contains no escaping and can contain newlines and any other characters:

``` kotlin
val text = """
  for (c in "foo")
    print(c)
"""
```


### Templates

Strings may contain template expressions, i.e. pieces of code that are evaluated and whose results are concatenated into the string. A template expression starts with a dollar sign ($) and consists of either a simple name:

``` kotlin
val i = 10
val s = "i = $i" // evaluates to "i = 10"
```

or an arbitrary expression in curly braces:

``` kotlin
val s = "abc"
val str = "$s.length is ${s.length}" // evaluates to "abc.length is 3"
```

