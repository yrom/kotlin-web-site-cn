---
type: doc
layout: reference
category: Basics
title: "编码规范"
---

# 编码规范

此页面包含当前 Kotlin 语言的编码风格

## 命名风格
如果拿不准的时候，默认使用Java的编码规范，比如：

* 使用驼峰法命名（并避免命名含有下划线）
* 类型名以大写字母开头
* 方法和属性以小写字母开头
* 使用 4 个空格缩进
* 公有函数应撰写函数文档，这样这些文档才会出现在 Kotlin Doc 中

## 冒号

类型和超类型之间的冒号前要有一个空格，而实例和类型之间的冒号前不要有空格：

``` kotlin
interface Foo<out T : Any> : Bar {
    fun foo(a: Int): T
}
```

## Lambda表达式

在lambda表达式中, 大括号左右要加空格，分隔参数与代码体的箭头左右也要加空格
。lambda表达应尽可能不要写在圆括号中

``` kotlin
list.filter { it > 10 }.map { element -> element * 2 }
```

在非嵌套的短lambda表达式中，最好使用约定俗成的默认参数 `it` 来替代显式声明参数名
。在嵌套的有参数的lambda表达式中，参数应该总是显式声明。

## Unit

如果函数返回 Unit 类型，该返回类型应该省略：

``` kotlin
fun foo() { // 省略了 ": Unit"

}
```

## Functions vs Properties

In some cases functions with no arguments might be interchangeable with read-only properties. 
Although the semantics are similar, there are some stylistic conventions on when to prefer one to another.

Prefer a property over a function when the underlying algorithm:

* does not throw
* has a `O(1)` complexity
* is cheap to calculate (or caсhed on the first run)
* returns the same result over invocations 
