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

## 类头格式化

有少数几个参数的类可以写成一行：

```kotlin 
class Person(id: Int, name: String)
```

具有较长类头的类应该这样格式化：每个主构造函数参数位于带有缩进的单独一行中。
此外，右括号应该另起一行。如果我们使用继承，那么超类构造函数调用或者实现接口列表
应位于与括号相同的行上：

```kotlin 
class Person(
    id: Int, 
    name: String,
    surname: String
) : Human(id, name) {
    // ……
}
```

对于多个接口，应首先放置超类构造函数调用，然后每个接口应位于不同的行中：

```kotlin 
class Person(
    id: Int, 
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker {
    // ……
}
```

构造函数参数可以使用常规缩进或连续缩进（双倍的常规缩进）。

## Unit

如果函数返回 Unit 类型，该返回类型应该省略：

``` kotlin
fun foo() { // 省略了 ": Unit"

}
```

## 函数还是属性

很多场合无参的函数可与只读属性互换。
尽管语义相近，也有一些取舍的风格约定。

底层算法优先使用属性而不是函数：

* 不会抛异常
* `O(1)` 复杂度
* 计算廉价（或缓存第一次运行）
* 不同调用返回相同结果 
