---
type: doc
layout: reference
category: "Classes and Objects"
title: "可见性修饰符"
---

# 可见性修饰符  

类，对象，接口，构造方法，和它们的setter方法都可以用_visibility modifiers_来做修饰。
（getter 总与属性有着相同的可见性。）
在Kotlin中有以下四个可见性修饰符：`private`、 `protected`、 `internal` 和 `public`。
默认可见性如果没有显式指定修饰符的话是 `public`。

下面将解释不同类型的声明范围。

## 包名

函数，属性和类，对象和接口可以在顶层声明，即直接在包内：

``` kotlin
// file name: example.kt
package foo

fun baz() {}
class Bar {}
```

*  如果你不指定任何可见性修饰符，那么默认情况下使用`public`修饰，这意味着你们将声明
随处可见;
* 如果你声明`private`，只会在声明它的文件内可见；
* 如果你声明`internal`，它会在相同模块内随处可见；
* `protected`不适用于顶层声明。

例子:

``` kotlin
// file name: example.kt
package foo

private fun foo() {} // visible inside example.kt

public var bar: Int = 5 // property is visible everywhere
    private set         // setter is visible only in example.kt
    
internal val baz = 6    // visible inside the same module
```

## 类和接口

当一个类中声明：

* `private` 意味着这个类只在内部可见(包含所有成员).
* `protected`--- 和`private`一样+在子类可见。
* `internal` --- 任何客户端 *inside this module* 谁看到声明类，其`internal`成员在里面;
* `public` ---  任何客户端看到声明类看到其`public`成员。

*注意* 对于Java用户:外部类不能访问Kotlin内部类的private成员。

If you override a `protected` member and do not specify the visibility explicitly, the overriding member will also have `protected` visibility.

例子:

``` kotlin
open class Outer {
    private val a = 1
    protected open val b = 2
    internal val c = 3
    val d = 4  // public by default
    
    protected class Nested {
        public val e: Int = 5
    }
}

class Subclass : Outer() {
    // a is not visible
    // b, c and d are visible
    // Nested and e are visible

    override val b = 5   // 'b' is protected
}

class Unrelated(o: Outer) {
    // o.a, o.b are not visible
    // o.c and o.d are visible (same module)
    // Outer.Nested is not visible, and Nested::e is not visible either 
}
```

### 构造函数

指定一个类的可见性的主构造函数,使用以下语法(注意你需要添加一个
显式 *构造函数* {:.keyword} keyword)：

``` kotlin
class C private constructor(a: Int) { ... }
```

这里的构造函数是私有的。不像其他的声明，在默认情况下，所有构造函数是`public`，这实际上
等于他们是随处可见，其中的类是可见(即内部类的构造函数是唯一
可见在同一模块内).

### 局部声明

局部变量，函数和类不能有可见性修饰符。


## Modules

The `internal` visibility modifier means that the member is visible with the same module. More specifically,
a module is a set of Kotlin files compiled together:

  * an IntelliJ IDEA module;
  * a Maven or Gradle project;
  * a set of files compiled with one invocation of the <kotlinc> Ant task.
