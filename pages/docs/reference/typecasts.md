---
type: doc
layout: reference
category: "Syntax"
title: "类型的检查与转换：“is”与“as”"
---

# 类型的检查与转换“is”与“as”

## `is` 与 `!is` 操作符

我们可以在运行时通过使用 `is` 操作符或其否定形式 `!is` 来检查对象是否符合给定类型：

``` kotlin
if (obj is String) {
    print(obj.length)
}

if (obj !is String) { // 与 !(obj is String) 相同
    print("Not a String")
}
else {
    print(obj.length)
}
```

## 智能转换

在许多情况下，不需要在 Kotlin 中使用显式转换操作符，因为编译器跟踪<!--
-->不可变值的 `is`-检查，并在需要时自动插入（安全的）转换：

``` kotlin
fun demo(x: Any) {
    if (x is String) {
        print(x.length) // x 自动转换为字符串
    }
}
```

编译器足够聪明，能够知道如果反向检查导致返回那么该转换是安全的：

``` kotlin
    if (x !is String) return
    print(x.length) // x 自动转换为字符串
```

或者在 `&&` 和 `||` 的右侧：

``` kotlin
    // `||` 右侧的 x 自动转换为字符串
    if (x !is String || x.length == 0) return

    // `&&` 右侧的 x 自动转换为字符串
    if (x is String && x.length > 0) {
        print(x.length) // x 自动转换为字符串
    }
```


这些 _智能转换_ 用于 [*when*{: .keyword }-表达式](control-flow.html#when-表达式)
和 [*while*{: .keyword }-循环 ](control-flow.html#while-循环) 也一样：

``` kotlin
when (x) {
    is Int -> print(x + 1)
    is String -> print(x.length + 1)
    is IntArray -> print(x.sum())
}
```

请注意，当编译器不能保证变量在检查和使用之间不可改变时，智能转换不能用。
更具体地，智能转换能否适用根据以下规则：

  * *val*{: .keyword } 局部变量——总是可以；
  * *val*{: .keyword } 属性——如果属性是 private 或 internal，或者该检查在声明属性的同一模块中执行。智能转换不适用于 open 的属性或者具有自定义 getter 的属性；
  * *var*{: .keyword } 局部变量——如果变量在检查和使用之间没有修改、并且没有在会修改它的 lambda 中捕获；
  * *var*{: .keyword } 属性——决不可能（因为该变量可以随时被其他代码修改）。

{:#不安全的转换操作符}

## “不安全的”转换操作符

通常，如果转换是不可能的，转换操作符会抛出一个异常。因此，我们称之为*不安全的*。
Kotlin 中的不安全转换由中缀操作符 *as*{: .keyword }（参见[operator precedence](grammar.html#precedence)）完成：

``` kotlin
val x: String = y as String
```

请注意，*null*{: .keyword } 不能转换为 `String` 因该类型不是[可空的](null-safety.html)，
即如果 `y` 为空，上面的代码会抛出一个异常。
为了匹配 Java 转换语义，我们必须在转换右边有可空类型，就像：

``` kotlin
val x: String? = y as String?
```

{:#安全的可空转换操作符}

## “安全的”（可空）转换操作符

为了避免抛出异常，可以使用*安全*转换操作符 *as?*{: .keyword }，它可以在失败时返回 *null*{: .keyword }：

``` kotlin
val x: String? = y as? String
```

请注意，尽管事实上 *as?*{: .keyword } 的右边是一个非空类型的 `String`，但是其转换的结果是可空的。
