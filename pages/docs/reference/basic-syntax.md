---
type: doc
layout: reference
category: "Basics"
title: "基本语法"
---

# 基本语法

## 定义包

包的声明应处于源文件顶部：

``` kotlin
package my.demo

import java.util.*

// ...
```

目录与包的结构无需匹配：源代码可以在文件系统的任意位置。

参见[包](packages.html)。

## 定义函数

带有两个 `Int` 参数、返回 `Int` 的函数：

``` kotlin
fun sum(a: Int, b: Int): Int {
    return a + b
}
```

将表达式作为函数体、返回值类型自动推断的函数：

``` kotlin
fun sum(a: Int, b: Int) = a + b
```

函数返回无意义的值：

``` kotlin
fun printSum(a: Int, b: Int): Unit {
    print(a + b)
}
```

`Unit` 返回类型可以省略：

``` kotlin
fun printSum(a: Int, b: Int) {
  print(a + b)
}
```

参见[函数](functions.html)。

## 定义局部变量

一次赋值（只读）的局部变量:

``` kotlin
val a: Int = 1
val b = 1  // 自动推断出 `Int` 类型
val c: Int // 如果没有初始值类型不能省略
c = 1      // 明确赋值
```

可变变量：

``` kotlin
var x = 5 // `自动推断出 Int` 类型
x += 1
```

参见[属性和字段](properties.html)。


## 注释

正如 Java 和 JavaScript，Kotlin 支持行注释及块注释。

``` kotlin
// 这是一个行注释

/* 这是一个多行的
   块注释。 */
```

与 Java 不同的是，Kotlin 的块注释可以嵌套。

参见[生成 Kotlin 代码文档](kotlin-doc.html) 查看关于文档注释语法的信息。

## 使用字符串模板

``` kotlin
fun main(args: Array<String>) {
    if (args.size == 0) return

    print("First argument: ${args[0]}")
}
```

参见[字符串模板](basic-types.html#字符串模板)。

## 使用条件表达式

``` kotlin
fun max(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}
```

使用 *if*{: .keyword } 作为表达式:

``` kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

参见[*if*{: .keyword } 表达式](control-flow.html#if表达式)。

## 使用可空值及 *null*{: .keyword } 检测

当某个变量的值可以为 *null*{: .keyword } 的时候，必须在声明处的类型后添加 `?` 来标识该引用可为空。

如果 `str` 的内容不是数字返回 *null*{: .keyword }：

``` kotlin
fun parseInt(str: String): Int? {
    // ...
}
```

使用返回可空值的函数:

``` kotlin
fun main(args: Array<String>) {
    if (args.size < 2) {
        print("Two integers expected")
        return
    }

    val x = parseInt(args[0])
    val y = parseInt(args[1])

    // 直接使用 `x * y` 可能会报错，因为他们可能为 null
    if (x != null && y != null) {
        // 在空检测后，x 和 y 会自动转换为非空值（non-nullable）
        print(x * y)
    }
}
```

或者

``` kotlin
    // ...
    if (x == null) {
        print("Wrong number format in '${args[0]}'")
        return
    }
    if (y == null) {
        print("Wrong number format in '${args[1]}'")
        return
    }

    // 在空检测后，x 和 y 会自动转换为非空值
    print(x * y)
```

参见[空安全](null-safety.html)。

## 使用类型检测及自动类型转换

*is*{: .keyword } 运算符检测一个表达式是否某类型的一个实例。
如果一个不可变的局部变量或属性已经判断出为某类型，那么检测后的分支中可以直接当作该类型使用，无需显式转换：

``` kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // `obj` 在该条件分支内自动转换成 `String`
        return obj.length
    }

    // 在离开类型检测分支后，`obj` 仍然是 `Any` 类型
    return null
}
```

或者

``` kotlin
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null

    // `obj` 在这一分支自动转换为 `String`
    return obj.length
}
```

甚至

``` kotlin
fun getStringLength(obj: Any): Int? {
    // `obj` 在 `&&` 右边自动转换成 `String` 类型
    if (obj is String && obj.length > 0) ｛
      return obj.length
    }

    return null
}
```

参见[类](classes.html) 和 [类型转换](typecasts.html)。

## 使用 `for` 循环

``` kotlin
fun main(args: Array<String>) {
    for (arg in args) {
        print(arg)
    } 
}
```

或者

``` kotlin
for (i in args.indices) {
    print(args[i])
} 
```

参见[for循环](control-flow.html#for循环)。

## Using a `while` loop

``` kotlin
fun main(args: Array<String>) {
    var i = 0
    while (i < args.size) {
        print(args[i++])
    }  
}
```

参见[while 循环](control-flow.html#while循环)。

## 使用 `when` 表达式

``` kotlin
fun cases(obj: Any) {
    when (obj) {
        1          -> print("One")
        "Hello"    -> print("Greeting")
        is Long    -> print("Long")
        !is String -> print("Not a string")
        else       -> print("Unknown")
    }
}
```

参见[when表达式](control-flow.html#when表达式)。

## 使用区间（range）

使用 *in*{: .keyword } 运算符来检测某个数字是否在指定区间内：

``` kotlin
if (x in 1..y-1) {
    print("OK")
}
```

检测某个数字是否在指定区间外:

``` kotlin
if (x !in 0..array.lastIndex) {
    print("Out")
}
```

区间内迭代:

``` kotlin
for (x in 1..5) {
    print(x)
}
```

参见[区间](ranges.html)。

## 使用集合

对集合进行迭代:

``` kotlin
for (name in names) {
    println(name)
}
```

使用 *in*{: .keyword } 运算符来判断集合内是否包含某实例：

``` kotlin
if (text in names) { // 会调用 names.contains(text)
    print("Yes")
}
```

使用 lambda 表达式来过滤（filter）和变换（map）集合：

``` kotlin
names
        .filter { it.startsWith("A") }
        .sortedBy { it }
        .map { it.toUpperCase() }
        .forEach { print(it) }
```

参见[高阶函数及Lambda表达式](lambdas.html)。
