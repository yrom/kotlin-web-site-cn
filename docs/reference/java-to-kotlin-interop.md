---
type: doc
layout: reference
category: "Interop"
title: "Java调用Kotlin代码"
---

# Java调用Kotlin代码

Java可以轻松调用Kotlin代码。

## 属性

属性getters被转换成 *get*-方法，setters转换成*set*-方法。

## 包级别的函数

`example.kt` 文件中 `org.foo.bar` 包内声明的所有的函数和属性，都会被放到一个叫`org.foo.bar.ExampleKt`的java类里。

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

可以使用 `@JvmName` 注解自定义生成的Java 类的类名：

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

如果多个文件中生成了相同的Java类名（包名相同，类名相同或者有相同的`@JvmName`注解）通常会报错，然而，可以在每个文件添加 `@JvmMultifileClass`注解，可以让编译器生成一个统一的带有特殊名字的类，这个类包含了对应这些文件中所有的声明。



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

## 实例字段

如果在 Java 需要像字段一样调用一个 Kotlin 的属性，你需要使用`@JvmField`注解。这个字段与属性具有相同的可见性。属性符合有实际字段(backing field)、非私有、没有`open`, `override` 或者 `const`修饰符、不是被委托的属性这些条件才可以使用`@JvmField`注解。


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

[延迟初始化](properties.html#late-initialized-properties) 的属性（在Java中）也可以被作为字段调用，字段的可见性和　`lateinit`　属性的　setter　相同。


## 静态字段

在一个命名对象或者伴生对象中声明的Koltin属性会持有静态实际字段(backing fields)，这些字段存在于该命名对象或者伴生对象中的。

通常，这些字段都是private的，但是他们可以通过以下方式暴露出来。

 - `@JvmField` 注解;
 - `lateinit` 修饰符;
 - `const` 修饰符.

用 `@JvmField` 注解该属性可以生成一个与该属性相同可见性的静态字段。

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

在命名对象或者伴生对象中的一个[延迟初始化](properties.html#late-initialized-properties) 的属性都有一个静态实际字段，字段和该属性的setter也有相同的可见性。


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

使用`const` 注解可以将 Kotlin 属性转换成 Java 中的静态字段。


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

## 静态方法

正如上面所说，Kotlin 自动为包级函数生成了静态方法
在Kotlin 中，还可以通过`@JvmStatic`注解在命名对象或者伴生对象中定义的函数来生成对应的静态方法。
例如：

``` kotlin
class C {
  companion object {
    @JustvmStatic fun foo() {}
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

通过使用　`@JvmStatic`　注解对象的属性或伴生对象，使对应的getter 和 setter 方法在这个对象或者包含这个伴生对象的类中也成为静态成员。




## 用@JvmName解决签名冲突

有时我们想让一个 Kotlin 里的命名函数在字节码里有另外一个 JVM 名字。
最突出的例子就是 *类名型擦除*:

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


## 生成重载

通常，如果你写一个有默认参数值的 Kotlin 方法，在 Java 里，只会有一个有完整参数的签名。如果你要暴露多个重载给java调用者，你可以使用
@JvmOverloads 注解。

``` kotlin
@JvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
    ...
}
```

对于每一个有默认值的参数，都会生成一个额外的重载，这个重载会把这个参数和它右边的所有参数都移除掉。在上面这个例子里，生成下面的方法：

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

## 可变泛型


当Kotlin 的类使用了 [declaration-site variance](generics.html#declaration-site-variance)，从Java 的角度看起来有两种用法，比如我们下面涉及到的这种用法的类和两个函数。


``` kotlin
class Box<out T>(val value: T)

interface Base
class Derived : Base

fun boxDerived(value: Derived): Box<Derived> = Box(value)
fun unboxBase(box: Box<Base>): Base = box.value
```

一种看似理所当然地将函数转换成 Java 代码的方式可能会是这样：

 
``` java
Box<Derived> boxDerived(Derived value) { ... }
Base unboxBase(Box<Base> box) { ... }
``` 

问题是，在Kotlin 中我们可以这样写 `unboxBase(boxDerived("s"))` ，但是这样的写法在 Java 中是无法通过的，因为在 Java 中`Box`的泛型参数 `T` 是*不可变的*，`Box<Derived>` 实际上并不是`Box<Base>`的子类。如果在 Java 中要编译通过我们需要像下面这样定义`unboxBase`：
[//]:# (The problem is that in Kotlin we can say , but in Java that would be impossible, because in Java the class `Box` is *invariant* in its parameter `T`, and thus `Box<Derived>` is not a subtype of `Box<Base>`.   To make it work in Java we'd have to define `unboxBase` as follows:)
  
``` java
Base unboxBase(Box<? extends Base> box) { ... }  
```  

我们在这里通过使用 Java 的 *通配符类型* (`? extends Base`) 去模拟[declaration-site variance](generics.html#declaration-site-variance)，因为在 Java   中只能这么做。

当作为参数的时候，为了让 Kotlin 的 API 工作，针对`Box` 我们将Koltin 中的 `Box<Super>` 在 Java 中生成为 `Box<? extends Super>`(`Foo`将生成为 `Foo<? super Bar>`) 。当作为返回值的时候，我们不需要生成通配符类型，因为如果生成通配符在 Java 中还需要做其他操作来转换(这是常见的 Java 代码风格)。因此上面例子中的函数实际上会被转换成下面的代码：

  
``` java
// 作为返回类型 - 没有泛型
Box<Derived> boxDerived(Derived value) { ... }
 
// 作为参数 - 带有泛型 
Base unboxBase(Box<? extends Base> box) { ... }
```

注意：如果参数类型是 final 的，就不用生成泛型了，比如，无论在什么地方`Box<String>`转换成 Java 代码始终还是　`Box<String>`，


如果我们不想要默认生成的通配符，需要自己指定可以使用`@JvmWildcard` 注解：


``` kotlin
fun boxDerived(value: Derived): Box<@JvmWildcard Derived> = Box(value)
// 将被转换成
// Box<? extends Derived> boxDerived(Derived value) { ... }
```

另一方面，如果我们根本不需要默认的通配符转换，我们可以使用`@JvmSuppressWildcards`


``` kotlin
fun unboxBase(box: Box<@JvmSuppressWildcards Base>): Base = box.value
// 将被转换成
// Base unboxBase(Box<Base> box) { ... }
```

注意：`@JvmSuppressWildcards` 不是只可以用在单独的类型参数上面，是可以是用在所有声明上，比如 函数，类等，其对应下面所有的泛型都不会自动转换为Java 中的通配符。



### Nothing 类型的转换
 
`Nothing` 是一种特殊的类型，因为它在 Java 中没有类型相对应。事实上，每个 Java 的引用类型，包括 `java.lang.Void` 都可以接受 `null`值，但是 Nothing 不行，因此在 Java 世界中没有什么可以代表这个类型，这就是为什么在Kotlin 中要生成原始类型需要使用 `Nothing`。

``` kotlin
fun emptyList(): List<Nothing> = listOf()
// 被转换为
// List emptyList() { ... }
```
