---
type: doc
layout: reference
category: "Interop"
title: "Calling Kotlin from Java"
---

# Java调用Kotlin代码

Java可以轻松调用Kotlin代码。

## 属性

属性getters被转换成 *get*-方法，setters转换成*set*-方法。

## 包级别的函数

`example.kt` 文件中 `org.foo.bar` 包内声明的所有的函数和属性，都会被放到一个
叫`org.foo.bar.ExampleKt`的java类里。

``` kotlin
// example.kt
package demo

class Foo

fun bar() {
}

```

``` java
// Java
new demo.Foo();
demo.ExampleKt.bar();
```

The name of the generated Java class can be changed using the `@JvmName` annotation:

``` kotlin
@file:JvmName("DemoUtils")

package demo

class Foo

fun bar() {
}

```

``` java
// Java
new demo.Foo();
demo.DemoUtils.bar();
```

Having multiple files which have the same generated Java class name (the same package and the same name or the same
@JvmName annotation) is normally an error. However, the compiler has the ability to generate a single Java facade
class which has the specified name and contains all the declarations from all the files which have that name.
To enable the generation of such a facade, use the @JvmMultifileClass annotation in all of the files.

``` kotlin
// oldutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun foo() {
}
```

``` kotlin
// newutils.kt
@file:JvmName("Utils")
@file:JvmMultifileClass

package demo

fun bar() {
}
```

``` java
// Java
demo.Utils.foo();
demo.Utils.bar();
```

## Instance Fields

If you need to expose a Kotlin property as a field in Java, you need to annotate it with the `@JvmField` annotation.
The field will have the same visibility as the underlying property. You can annotate a property with `@JvmField`
if it has a backing field, is not private, does not have `open`, `override` or `const` modifiers, and is not a delegated property.

``` kotlin
class C(id: String) {
    @JvmField val ID = id
}
```

``` java
// Java
class JavaClient {
    public String getID(C c) {
        return c.ID;
    }
}
```

[Late-Initialized](properties.html#late-initialized-properties) properties are also exposed as fields. 
The visibility of the field will be the same as the visibility of `lateinit` property setter.

## 静态字段

Kotlin properties declared in a named object or a companion object will have static backing fields
either in that named object or in the class containing the companion object.

Usually these fields are private but they can be exposed in one of the following ways:

 - `@JvmField` annotation;
 - `lateinit` modifier;
 - `const` modifier.
 
Annotating such a property with `@JvmField` makes it a static field with the same visibility as the property itself.

``` kotlin
class Key(val value: Int) {
    companion object {
        @JvmField
        val COMPARATOR: Comparator<Key> = compareBy<Key> { it.value }
    }
}
```

``` java
// Java
Key.COMPARATOR.compare(key1, key2);
// public static final field in Key class
```

A [late-initialized](properties.html#late-initialized-properties) property in an object or a companion object
has a static backing field with the same visibility as the property setter.

``` kotlin
object Singleton {
    lateinit var provider: Provider
}
```

``` java
// Java
Singleton.provider = new Provider();
// public static non-final field in Singleton class
```

Properties annotated with `const` (in classes as well as at the top level) are turned into static fields in Java:

``` kotlin
// file example.kt

object Obj {
  const val CONST = 1
}

class C {
    companion object {
        const val VERSION = 9
    }
}

const val MAX = 239
```

In Java:

``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
int v = C.VERSION;
```

## Static Methods

As mentioned above, Kotlin generates static methods for package-level functions.
Kotlin can also generate static methods for functions defined in named objects or companion objects if you annotate those functions as `@JvmStatic`.
For example:

``` kotlin
class C {
  companion object {
    @JvmStatic fun foo() {}
    fun bar() {}
  }
}
```

现在，`foo()`在java里就是静态的了，而`bar()` 不是：

``` java
C.foo(); // 没问题
C.bar(); // 错误: 不是一个静态方法
```

同样的，命名对象：

``` kotlin
object Obj {
    @JvmStatic fun foo() {}
    fun bar() {}
}
```

Java 里：

``` java
Obj.foo(); // 没问题
Obj.bar(); // 错误
Obj.INSTANCE.bar(); // 对单例的方法调用
Obj.INSTANCE.foo(); // 也行
```

`@JvmStatic` annotation can also be applied on a property of an object or a companion object
making its getter and setter methods be static members in that object or the class containing the companion object.


## 用@JvmName解决签名冲突

有时我们想让一个 Kotlin 里的命名函数在字节码里有另外一个 JVM 名字。
最突出的例子来自于 *类型擦除*:

``` kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```

这两个函数不能同时定义，因为它们的 JVM 签名是一样的：`filterValid(Ljava/util/List;)Ljava/util/List;`.
如果我们真的相让它们在 Kotlin里用同一个名字，我们需要用`@JvmName`去注释它们中的一个（或两个），指定的另外一个名字当参数：

``` kotlin
fun List<String>.filterValid(): List<String>

@JvmName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```

在Kotlin里它们可以都用`filterValid`来访问，但是在Java里，它们是`filterValid` 和 `filterValidInt`.

同样的技巧也适用于属性 `x` 和函数 `getX()` 共存：

``` kotlin
val x: Int
  @JvmName("getX_prop")
  get() = 15

fun getX() = 10
```


## 重载生成

通常，如果你写一个有默认参数值的 Kotlin 方法，在 Java 里，只会有一个
有完整参数的签名。如果你要暴露多个重载给java调用者，你可以使用
@JvmOverloads 注解。

``` kotlin
@JvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
    ...
}
```

对于每一个有默认值的参数，都会生成一个额外的重载，这个重载会把这个参数
和它右边的所有参数都移除掉。在上面这个例子里，生成下面的方法：

``` java
// Java
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```

构造函数，静态函数等也能用这个标记。但他不能用在抽象方法上，包括
接口中的方法。

注意一下，[Secondary Constructors](classes.html#secondary-constructors) 描述过，如果一个类的所有构造函数参数都有默认
值，会生成一个公开的无参构造函数。这就算
没有@JvmOverloads 注解也有效。


## 受检异常

上面说过，kotlin没有受检异常。
所以，通常，kotlin函数的java签名没有声明抛出异常。
于是如果我们有一个kotlin函数：

``` kotlin
// example.kt
package demo

fun foo() {
  throw IOException()
}
```

然后我们想要在java里调用它，捕捉这个异常：

``` java
// Java
try {
  demo.Example.foo();
}
catch (IOException e) { // 错误: foo() 没有声明 IOException
  // ...
}
```

因为`foo()`没有声明 `IOException`，java编译器报了错误信息。
为了解决这个问题，要在kotlin里使用`@throws`标记。

``` kotlin
@Throws(IOException::class)
fun foo() {
    throw IOException()
}
```

## Null安全性

当从 Java 中调用 Kotlin 函数时，没人阻止我们传递 *null*{: .keyword } 给一个非空参数。
这就是为什么 Kotlin 给所有期望非空参数的公开函数生成运行时检测。
这样我们就能在 Java 代码里立即得到 `NullPointerException`。

## Variant generics

When Kotlin classes make use of [declaration-site variance](generics.html#declaration-site-variance), there are two 
options of how their usages are seen from the Java code. Let's say we have the following class and two functions that use it:

``` kotlin
class Box<out T>(val value: T)

interface Base
class Derived : Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```

A naive way of translating these functions into Java would be this:
 
``` java
Box<Derived> boxDerived(Derived value) { ... }
Base unboxBase(Box<Base> box) { ... }
``` 

The problem is that in Kotlin we can say `unboxBase(boxDerived("s"))`, but in Java that would be impossible, because in Java 
  the class `Box` is *invariant* in its parameter `T`, and thus `Box<Derived>` is not a subtype of `Box<Base>`. 
  To make it work in Java we'd have to define `unboxBase` as follows:
  
``` java
Base unboxBase(Box<? extends Base> box) { ... }  
```  

Here we make use of Java's *wildcards types* (`? extends Base`) to emulate declaration-site variance through use-site 
variance, because it is all Java has.

To make Kotlin APIs work in Java we generate `Box<Super>` as `Box<? extends Super>` for covariantly defined `Box` 
(or `Foo<? super Bar>` for contravariantly defined `Foo`) when it appears *as a parameter*. When it's a return value,
we don't generate wildcards, because otherwise Java clients will have to deal with them (and it's against the common 
Java coding style). Therefore, the functions from our example are actually translated as follows:
  
``` java
// return type - no wildcards
Box<Derived> boxDerived(Derived value) { ... }
 
// parameter - wildcards 
Base unboxBase(Box<? extends Base> box) { ... }
```

NOTE: when the argument type is final, there's usually no point in generating the wildcard, so `Box<String>` is always
  `Box<String>`, no matter what position it takes.

If we need wildcards where they are not generated by default, we can use the `@JvmWildcard` annotation:

``` kotlin
fun boxDerived(value: Derived): Box<@JvmWildcard Derived> = Box(value)
// is translated to 
// Box<? extends Derived> boxDerived(Derived value) { ... }
```

On the other hand, if we don't need wildcards where they are generated, we can use `@JvmSuppressWildcards`:

``` kotlin
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
// is translated to 
// Base unboxBase(Box<Base> box) { ... }
```

NOTE: `@JvmSuppressWildcards` can be used not only on individual type arguments, but on entire declarations, such as 
functions or classes, causing all wildcards inside them to be suppressed.

### Translation of type Nothing
 
The type `Nothing` is special, because it has no natural counterpart in Java. Indeed, every Java reference type, including
`java.lang.Void`, accepts `null` as a value, and `Nothing` doesn't accept even that. So, this type cannot be accurately
represented in the Java world. This is why Kotlin generates a raw type where an argument of type `Nothing` is used:

``` kotlin
fun emptyList(): List<Nothing> = listOf()
// is translated to
// List emptyList() { ... }
```
