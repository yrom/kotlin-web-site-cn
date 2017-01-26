---
type: doc
layout: reference
category: "JavaScript"
title: "JavaScript 反射"
---

# JavaScript 反射

在 Kotlin 编译成的 JavaScript 中，有一个属性
可用于任何对象，名为 `jsClass`，它返回一个 `JsClass` 实例。`JsClass` 目前只能提供
（无限定的）类的名称。然而，`JsClass` 实例本身是一个对构造函数的引用。
这可以用于与期待构造函数的引用的 JS 函数互操作。

要获得对类的引用，可以使用 `::class` 语法。Kotlin for JavaScript 目前不支持完整的反射 API；
唯一可用的属性是 `.simpleName` 返回类的名称
以及 `.js` 返回相应的`JsClass`。

示例：

``` kotlin
class A
class B
class C

inline fun <reified T> foo() {
    println(jsClass<T>().name)
}

println(A().jsClass.name)     // 输出“A”
println(B::class.simpleName)  // 输出“B”
println(B::class.js.name)     // 输出“B”
foo<C>()                      // 输出“C”
```
