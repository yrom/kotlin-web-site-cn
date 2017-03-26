---
type: doc
layout: reference
category: "Syntax"
title: "动态类型"
---

# 动态类型

> 在面向 JVM 平台的代码中不支持动态类型
{:.note}

作为一种静态类型的语言，Kotlin仍然需要与无类型或松散类型的环境（例如
JavaScript生态系统）进行互操作。为了方便这些使用场景，语言中有 `dynamic` 类型可用：

``` kotlin
val dyn: dynamic = ……
```

`dynamic` 类型基本上关闭了 Kotlin 的类型检查系统：

  - 该类型的值可以赋值给任何变量或作为参数传递到任何位置，
  - 任何值都可以赋值给 `dynamic` 类型的变量，或者传递给一个接受 `dynamic` 作为参数的函数，
  - `null`-检查对这些值是禁用的。

`dynamic` 最特别的特性是，我们可以对 `dynamic` 变量调用**任何**属性或以任意参数调用**任何**函数
：

``` kotlin
dyn.whatever(1, "foo", dyn) // 'whatever' 在任何地方都没有定义
dyn.whatever(*arrayOf(1, 2, 3))
```

在 JavaScript 平台上，该代码将按照原样编译：在生成的 JavaScript 代码中，Kotlin中的 `dyn.whatever(1)` 变为 `dyn.whatever(1)`
。

When calling functions written in Kotlin on values of `dynamic` type, keep in mind the name mangling performed by the
Kotlin to JavaScript compiler. You may need to use the [@JsName annotation](js-to-kotlin-interop.html#jsname-注解)
to assign well-defined names to the functions that you need to call.

动态调用总是返回 `dynamic` 作为结果，所以我们可以自由地这样链接调用：

``` kotlin
dyn.foo().bar.baz()
```

当我们把一个 lambda 表达式传给一个动态调用时，它的所有参数默认都是 `dynamic` 类型的：

``` kotlin
dyn.foo {
    x -> x.bar() // x 是 dynamic
}
```

Expressions using values of `dynamic` type are translated to JavaScript "as is", and do not use the Kotlin operator conventions.
The following operators are supported:

* binary: `+`, `-`, `*`, `/`, `%`, `>`, `<` `>=`, `<=`, `==`, `!=`, `===`, `!==`, `&&`, `||`
* unary
    * prefix: `-`, `+`, `!`
    * prefix and postfix: `++`, `--`
* assignments: `+=`, `-=`, `*=`, `/=`, `%=`
* indexed access:
    * read: `d[a]`, more than one argument is an error
    * write: `d[a1] = a2`, more than one argument in `[]` is an error

`in`, `!in` and `..` operations with values of type `dynamic` are forbidden.

更多技术说明请参见[规范文档](https://github.com/JetBrains/kotlin/blob/master/spec-docs/dynamic-types.md)。


