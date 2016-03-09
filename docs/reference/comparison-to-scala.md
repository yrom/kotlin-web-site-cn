---
type: doc
layout: reference
category: FAQ
title: "Comparison to Scala"
---

# 对比Scala

The main goal of the Kotlin team is to create a pragmatic and productive programming language, rather than to advance the state of the art in programming language research.
稍作考虑, 如果你对Scala已相当得心应手,你或许不需要再学习Kotlin

## Scala有什么Kotlin没有的

* 隐式转换, 限定性因素, 等等
    * 在Scala中, 由于画面中有太多的隐式转换，有时不使用debugger,会很难弄清code中具体发生了什么
    * 为了做到功能多样性，Kotlin会使用[扩展函数](extensions.html).
* 重写类型成员
* 路径依赖性类型
* 宏
* 存在类型
    * [类型预测](generics.html#type-projections) 是一种非常特殊的情况
* 特性初始化的复杂逻辑
    * 参照 [类和接口](classes.html)
* 自定义符号运算
    * 参照 [运算符重载](operator-overloading.html)
* 嵌入式 XML
    * 参照 [类型安全 Groovy风格 builders](type-safe-builders.html)
* 结构类型
* 值类型
    * We plan to support [Project Valhalla](http://openjdk.java.net/projects/valhalla/) once it is released as part of the JDK
* Yield operator
* Actors
    * Kotlin supports [Quasar](http://www.paralleluniverse.co/quasar/), a third-party framework for actor support on the JVM
* 并行集合
    * Kotlin supports Java 8 streams, which provide similar functionality

## Kotlin有什么Scala没有的

* [零开销 空安全](null-safety.html)
    * Scala 的 Option，它是语法和运行时包装
* [智能转换](typecasts.html)
* [Kotlin的内联函数助力非局部跳跃](inline-functions.html#inline-functions)
* [第一类 授权](delegation.html). Also implemented via 3rd party plugin: Autoproxy
* [Member references](reflection.html#function-references) (also supported in Java 8).
