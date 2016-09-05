---
type: doc
layout: reference
category: "Syntax"
title: "接口"
---

# 接口

Kotlin 的接口与 Java 8 类似，既包含抽象方法的声明，也包含
实现。与抽象类不同的是，接口无法保存状态。它可以有
属性但必须声明为 abstract或提供访问器实现。

使用关键字 *interface*{: .keyword } 来定义接口。

``` kotlin
interface MyInterface {
    fun bar()
    fun foo() {
      // optional body
    }
}
```

## 实现接口

一个类或者对象可以实现一个或多个接口。

``` kotlin
class Child : MyInterface {
   override fun bar() {
      // body
   }
}
```

## 接口属性

你可以在接口定义属性。在接口中声明的属性要么是抽象的，要么提供访问器的实现。在接口中声明的属性不能有幕后字段（backing field），因此访问器不能引用他们。

``` kotlin
interface MyInterface {
    val property: Int // abstract

    val propertyWithImplementation: String
        get() = "foo"

    fun foo() {
        print(property)
    }
}

class Child : MyInterface {
    override val property: Int = 29
}
```

## 解决重写（Override）冲突

实现多个接口时，可能会遇到接口方法名同名的问题。

``` kotlin
interface A {
  fun foo() { print("A") }
  fun bar()
}

interface B {
  fun foo() { print("B") }
  fun bar() { print("bar") }
}

class C : A {
  override fun bar() { print("bar") }
}

class D : A, B {
  override fun foo() {
    super<A>.foo()
    super<B>.foo()
  }
}
```

上例中，接口 *A* 和 *B* 都定义了方法 *foo()* 和 *bar()*。 两者都实现了 *foo()*, 但是只有 *B* 实现了 *bar()* (*bar()* 在 *A* 中没有标记为 abstract，
因为没有方法体时默认为 abstract）。因为 *C* 是一个实现了 *A* 的具体类，所以必须要重写 *bar()* 并
实现这个抽象方法。*D* 可以不用重写 *bar()*，因为它实现了 *A* 和 *B*，因而可以自动继承 *B* 中 *bar()* 的实现，
但是两个接口都定义了方法 *foo()*，为了告诉编译器 *D* 会继承谁的方法，必须在 *D* 中重写 *foo()*。
