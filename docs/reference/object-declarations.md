---
type: doc
layout: reference
category: "Syntax"
title: "Object Expressions and Declarations"
---

# Object Expressions and Declarations 对象表达和对象声明

Sometimes we need to create an object of a slight modification of some class, without explicitly declaring a new subclass for it.

有些时候我们需要创造一个对象对某些类做稍微改变，而不用为了它明确定义一个新的子类。

Java handles this case with *anonymous inner classes*.
Kotlin slightly generalizes this concept with *object expressions* and *object declarations*.

Java把这处理为*匿名内部类*。在Kotlin稍微归纳为*对象表达*和*对象声明*。
## Object expressions 对象表达

To create an object of an anonymous class that inherits from some type (or types), we write:

创建一个继承自一些类型的内部类的对象，我们可以这么写：

``` kotlin
window.addMouseListener(object : MouseAdapter() {
  override fun mouseClicked(e: MouseEvent) {
    // ...
  }

  override fun mouseEntered(e: MouseEvent) {
    // ...
  }
})
```

If a supertype has a constructor, appropriate constructor parameters must be passed to it.
Many supertypes may be specified as a comma-separated list after the colon:

如果父类型有一个构造函数，合适的构造函数参数必须被传递下去。多父类型需要在冒号后被逗号隔开提出：


``` kotlin
open class A(x: Int) {
  public open val y: Int = x
}

interface B {...}

val ab = object : A(1), B {
  override val y = 15
}
```

If, by any chance, we need "just an object", with no nontrivial supertypes, we can simply say:

或许如果我们需要“仅仅是一个对象”,它没有非平凡的父类，我们可以简单这么写：

``` kotlin
val adHoc = object {
  var x: Int = 0
  var y: Int = 0
}
print(adHoc.x + adHoc.y)
```

Just like Java's anonymous inner classes, code in object expressions can access variables from the enclosing scope.

就像Jave的匿名内部类，在对象表达里代码可以使变量与作用域联系起来

(Unlike Java, this is not restricted to final variables.)

（与Java不同的是，这不是受final变量限制的。）

``` kotlin
fun countClicks(window: JComponent) {
  var clickCount = 0
  var enterCount = 0

  window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
      clickCount++
    }

    override fun mouseEntered(e: MouseEvent) {
      enterCount++
    }
  })
  // ...
}
```

## Object declarations 对象声明

[Singleton](http://en.wikipedia.org/wiki/Singleton_pattern) is a very useful pattern, and Kotlin (after Scala) makes it easy to declare singletons:

[单例模式]是一种非常有用的模式，而在Kotilin（在Scala之后）中使得单例模式很容易声明。

``` kotlin
object DataProviderManager {
  fun registerDataProvider(provider: DataProvider) {
    // ...
  }

  val allDataProviders : Collection<DataProvider>
    get() = // ...
}
```

This is called an *object declaration*. If there's a name following the *object*{: .keyword } keyword, we are not talking about an _expression_ anymore.
We cannot assign such a thing to a variable, but we can refer to it by its name. Such objects can have supertypes:

这被称为*对象声明*。如果有一个*object*{: .keyword }关键字在名字前面，这不能再被称为_表达_。我们不能把它归于变量，但我们可以通过它的名字来指定它。这些对象可以有父类型：

``` kotlin
object DefaultListener : MouseAdapter() {
  override fun mouseClicked(e: MouseEvent) {
    // ...
  }

  override fun mouseEntered(e: MouseEvent) {
    // ...
  }
}
```

**NOTE**: object declarations can't be local (i.e. be nested in directly inside a function), but they can be nested into other object declarations or non-inner classes.


**NOTE**: 对象声明不能是本地的（例如：直接嵌套在函数里面），但它们可以被嵌套进另外的对象声明或者非内部类里。


### Companion Objects 伴生对象

An object declaration inside a class can be marked with the *companion*{: .keyword } keyword:

一个对象声明在一个类里可以标志上*companion*{: .keyword }这个关键字：

``` kotlin
class MyClass {
  companion object Factory {
    fun create(): MyClass = MyClass()
  }
}
```

Members of the companion object can be called by using simply the class name as the qualifier:

伴生对象的成员可以被称为使用简单的类名称作为限定符：

``` kotlin
val instance = MyClass.create()
```

The name of the companion object can be omitted, in which case the name `Companion` will be used:

伴生对象的名字可以被省略，这种情况下`Companion`会被使用到：

``` kotlin
class MyClass {
  companion object {
  }
}

val x = MyClass.Companion
```

Note that, even though the members of companion objects look like static members in other languages, at runtime those
are still instance members of real objects, and can, for example, implement interfaces:

记住，虽然伴生对象的成员在其他语言中看起来像静态成员，但在日常使用中它们仍然是实体的实例成员，而且比如说能继承接口：


``` kotlin
interface Factory<T> {
  fun create(): T
}


class MyClass {
  companion object : Factory<MyClass> {
    override fun create(): MyClass = MyClass()
  }
}
```

However, on the JVM you can have members of companion objects generated as real static methods and fields, if you use
the `@platformStatic` annotation. See the [Java interoperability](java-interop.html#static-methods-and-fields) section
for more details.

然而，在JVM中你可以有些产生自真正的静态方法和域的伴生对象的成员，如果你使用`@platformStatic`注释。可以从[Java interoperability](java-interop.html#static-methods-and-fields) 这里查看详情。




### Semantical difference between object expressions and declarations 对象表达与对象声明语义上的不同

There is one important semantical difference between object expressions and object declarations:

这是一个重要的的不同在对象表达与对象声明语义上

* object declarations are initialized **lazily**, when accessed for the first time
* 对象声明被**懒散地**初始化，当被第一次访问的时候
* object expressions are executed (and initialized) **immediately**, where they are used
* 对对象表达被**立即**执行（被初始化），当它被用到的时候


