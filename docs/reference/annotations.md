---
type: doc
layout: reference
category: "Syntax"
title: "注解"
---

# 注解

## 注解的声明

注解是连接元数据以及代码的。为了声明注解，把*annotation*{: .keyword } 这个关键字放在类前面：

``` kotlin
annotation class Fancy
```

Additional attributes of the annotation can be specified by annotating the annotation class with meta-annotations:

  * [`@Target`](/api/latest/jvm/stdlib/kotlin.annotation/-target/index.html) specifies the possible kinds of
    elements which can be annotated with the annotation (classes, functions, properties, expressions etc.);
  * [`@Retention`](/api/latest/jvm/stdlib/kotlin.annotation/-retention/index.html) specifies whether the
    annotation is stored in the compiled class files and whether it's visible through reflection at runtime
    (by default, both are true);
  * [`@Repeatable`](/api/latest/jvm/stdlib/kotlin.annotation/-repeatable/index.html) allows using the same annotation
    on a single element multiple times;
  * [`@MustBeDocumented`](/api/latest/jvm/stdlib/kotlin.annotation/-must-be-documented/index.html) specifies that the
    annotation is part of the public API and should be included in the class or method signature shown in the
    generated API documentation.

``` kotlin
@Target(AnnotationTarget.CLASS, AnnotationTarget.FUNCTION,
        AnnotationTarget.VALUE_PARAMETER, AnnotationTarget.EXPRESSION)
@Retention(AnnotationRetention.SOURCE)
@MustBeDocumented
public annotation class Fancy
```

### 用途

``` kotlin
@Fancy class Foo {
  @Fancy fun baz(@Fancy foo: Int): Int {
    return (@Fancy 1)
  }
}
```


> ~~在很多情况下，`@ `这个标志不是强制性使用的。它只是在当注解表达式或者本地声明时需要：~~
>
> ``` kotlin
> fancy class Foo {
>   fancy fun baz(fancy foo: Int): Int {
>     @fancy fun bar() { ... }
>     return (@fancy 1)
>   }
> }
> ```


如果你需要注解类的主构造方法，你需要给构造方法的声明添加*constructor*{: .keyword}这个关键字，还有在前面添加注解：


``` kotlin
class Foo @Inject constructor(dependency: MyDependency) {
  // ...
}
```

你也可以注解属性访问器：

``` kotlin
class Foo {
    var x: MyDependency? = null
        @Inject set
}
```

### 构造方法

注解可以有参数的构造方法。

``` kotlin
annotation class Special(val why: String)

@Special("example") class Foo {}
```

Allowed parameter types are:

 * types that correspond to Java primitive types (Int, Long etc.);
 * strings;
 * classes (`Foo::class`);
 * enums;
 * other annotations;
 * arrays of the types listed above.

If an annotation is used as a parameter of another annotation, its name is not prefixed with the @ character:

``` kotlin
public annotation class ReplaceWith(val expression: String)

public annotation class Deprecated(
        val message: String,
        val replaceWith: ReplaceWith = ReplaceWith(""))

@Deprecated("This function is deprecated, use === instead", ReplaceWith("this === other"))
```

### Lambdas

注解也可以用在lambda表达式中。这将会应用到 lambda 生成的`invoke()`方法。这对[Quasar](http://www.paralleluniverse.co/quasar/)框架很有用，在这个框架中注解被用来并发控制

``` kotlin
annotation class Suspendable

val f = @Suspendable { Fiber.sleep(10) }
```

## Annotation Use-site Targets

When you're annotating a property or a primary constructor parameter, there are multiple Java elements which are
generated from the corresponding Kotlin element, and therefore multiple possible locations for the annotation in
the generated Java bytecode. To specify how exactly the annotation should be generated, use the following syntax:

``` kotlin
class Example(@field:Ann val foo,    // annotate Java field
              @get:Ann val bar,      // annotate Java getter
              @param:Ann val quux)   // annotate Java constructor parameter
```

The same syntax can be used to annotate the entire file. To do this, put an annotation with the target `file` at
the top level of a file, before the package directive or before all imports if the file is in the default package:

``` kotlin
@file:JvmName("Foo")

package org.jetbrains.demo
```

If you have multiple annotations with the same target, you can avoid repeating the target by adding brackets after the
target and putting all the annotations inside the brackets:

``` kotlin
class Example {
     @set:[Inject VisibleForTesting]
     public var collaborator: Collaborator
}
```

The full list of supported use-site targets is:

  * `file`
  * `property` (annotations with this target are not visible to Java)
  * `field`
  * `get` (property getter)
  * `set` (property setter)
  * `receiver` (receiver parameter of an extension function or property)
  * `param` (constructor parameter)
  * `setparam` (property setter parameter)
  * `delegate` (the field storing the delegate instance for a delegated property)

To annotate the receiver parameter of an extension function, use the following syntax:

``` kotlin
fun @receiver:Fancy String.myExtension() { }
```

If you don't specify a use-site target, the target is chosen according to the `@Target` annotation of the annotation
being used. If there are multiple applicable targets, the first applicable target from the following list is used:

  * `param`
  * `property`
  * `field`


## Jave注解

Java注解是百分百适用于Kotlin：

``` kotlin
import org.junit.Test
import org.junit.Assert.*

class Tests {
  @Test fun simple() {
    assertEquals(42, getTheAnswer())
  }
}
```


> ~~Java注解也可像用import修饰符重新命名：~~
>
> ``` kotlin
> import org.junit.Test as test
>
> class Tests {
>   test fun simple() {
>     ...
>   }
> }
> ```

因为在Java里，注释的参数顺序不是明确的，你不能使用常规的方法
调用语法传递的参数。相反的，你需要使用指定的参数语法。

``` java
// Java
public @interface Ann {
    int intValue();
    String stringValue();
}
```

``` kotlin
// Kotlin
@Ann(intValue = 1, stringValue = "abc") class C
```

就像在Jave里一样，需要一个特殊的参数是' value`参数;它的值可以使用不明确的名称来指定。

``` java
// Java
public @interface AnnWithValue {
    String value();
}
```

``` kotlin
// Kotlin
@AnnWithValue("abc") class C
```

如果在Java中`value`参数是array类型，在Kotlin中必须使用 `vararg`这个参数。

``` java
// Java
public @interface AnnWithArrayValue {
    String[] value();
}
```

``` kotlin
// Kotlin
@AnnWithArrayValue("abc", "foo", "bar") class C
```

如果你需要像注解参数一样指定一个类，使用一个Kotlin的类吧([KClass](/api/latest/jvm/stdlib/kotlin.reflect/-k-class/index.html))。Kotlin编译器会自动把它转换成Java类，使得Java代码能正常看到注解和参数。

``` kotlin

import kotlin.reflect.KClass

annotation class Ann(val arg1: KClass<*>, val arg2: KClass<out Any?>)

@Ann(String::class, Int::class) class MyClass
```

注解实例的值被视为Kotlin的属性。

``` java
// Java
public @interface Ann {
    int value();
}
```

``` kotlin
// Kotlin
fun foo(ann: Ann) {
    val i = ann.value
}
```


