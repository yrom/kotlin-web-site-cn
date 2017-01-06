---
type: doc
layout: reference
category: "Syntax"
title: "委托"
---

# 委托

## 类委托

[委托模式](https://zh.wikipedia.org/wiki/%E5%A7%94%E6%89%98%E6%A8%A1%E5%BC%8F)已经证明是实现继承的一个很好的替代方式，
而 Kotlin 可以零样板代码地原生支持它。
类 `Derived` 可以继承一个接口 `Base`，并将其所有共有的方法委托给一个指定的对象：

``` kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main(args: Array<String>) {
    val b = BaseImpl(10)
    Derived(b).print() // 输出 10
}
```

`Derived` 的超类型列表中的 *by*{: .keyword }-子句表示 `b` 将会在 `Derived` 中内部存储。
并且编译器将生成转发给 `b` 的所有 `Base` 的方法。

