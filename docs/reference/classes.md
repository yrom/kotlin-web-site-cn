---
type: doc
layout: reference
category: "Syntax"
title: "Classes and Inheritance"
related:
    - functions.md
    - nested-classes.md
    - interfaces.md
---

# Classes and Inheritance

## Classes

Classes in Kotlin are declared using the keyword *class*{: .keyword }:
类声明Kotlin使用关键字*class *{:.keyword}

``` kotlin
class Invoice {
}
```

The class declaration consists of the class name, the class header (specifying its type parameters, the primary
constructor etc.) and the class body, surrounded by curly braces. Both the header and the body are optional;
if the class has no body, curly braces can be omitted.

这个类声明被花括号包围，包括类名、类头(指定其类型参数,主构造函数等)和这个类的主干。类头和主干都是可选的；
如果这个类没有主干，花括号可以被省略。

``` kotlin
class Empty
```


### Constructors 
构造

A class in Kotlin can have a **primary constructor** and one or more **secondary constructors**. The primary
constructor is part of the class header: it goes after the class name (and optional type parameters).

在Kotlin中的类可以有**主构造函数**和一个或多个**二级构造函数**。主构造函数是类头的一部分:它运行在这个类名后面（和可选的类型参数）

``` kotlin
class Person constructor(firstName: String) {
}
```

If the primary constructor does not have any annotations or visibility modifiers, the *constructor*{: .keyword }
keyword can be omitted:
如果这个主构造函数不能有任何注释或者可见的修饰符，这个*constructor*{: .keyword }关键字可以被省略


``` kotlin
class Person(firstName: String) {
}
```

The primary constructor cannot contain any code. Initialization code can be placed
in **initializer blocks**, which are prefixed with the *init*{: .keyword }:

这个主构造函数不能包含任何的代码。初始化的代码可以被放置在**initializer blocks**（初始的模块），以*init*为前缀作为关键字{:.keyword}

``` kotlin
class Customer(name: String) {
    init {
        logger.info("Customer initialized with value ${name}")
    }
}
```

Note that parameters of the primary constructor can be used in the initializer blocks. They can also be used in
property initializers declared in the class body:
请注意，主构造的参数可以在初始化模块中使用。它们也可以在类体内声明初始化的属性：

``` kotlin
class Customer(name: String) {
    val customerKey = name.toUpperCase()
}
```

In fact, for declaring properties and initializing them from the primary constructor, Kotlin has a concise syntax:
事实上，声明属性和初始化主构造函数,Kotlin有简洁的语法:

``` kotlin
class Person(val firstName: String, val lastName: String, var age: Int) {
  // ...
}
```

Much the same way as regular properties, the properties declared in the primary constructor can be
mutable (*var*{: .keyword }) or read-only (*val*{: .keyword }).
一样普通属性,主构造函数中声明的属性可以是可变的或者是只读的

If the constructor has annotations or visibility modifiers, the *constructor*{: .keyword } keyword is required, and
the modifiers go before it:
如果构造函数注释或可见性修饰符，这个*constructor*{: .keyword }关键字是被需要的，这个修饰符运行在它前面

``` kotlin
class Customer public inject constructor(name: String) { ... }
```

For more details, see [Visibility Modifiers](visibility-modifiers.html#constructors).
为了关注更多的细节，看[Visibility Modifiers](visibility-modifiers.html#constructors)

#### Secondary Constructors
二级构造函数

The class can also declare **secondary constructors**, which are prefixed with *constructor*{: .keyword }:
这个类也可以被称为"二级构造函数"，通常被加上前缀"constructor"

``` kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

If the class has a primary constructor, each secondary constructor needs to delegate to the primary constructor, either
directly or indirectly through another secondary constructor(s). Delegation to another constructor of the same class
is done using the *this*{: .keyword } keyword:
如果类有一个主构造函数,每个二级构造函数需要委托给主构造函数,直接或间接地通过另一个二级函数。
委托到另一个使用同一个类的构造函数用"this"关键字

``` kotlin
class Person(val name: String) {
    constructor(name: String, parent: Person) : this(name) {
        parent.children.add(this)
    }
}
```

If a non-abstract class does not declare any constructors (primary or secondary), it will have a generated primary
constructor with no arguments. The visibility of the constructor will be public. If you do not want your class
to have a public constructor, you need to declare an empty primary constructor with non-default visibility:
如果一个非抽象类没有声明任何构造函数（原发性或继发性），这将有一个生成的主构造函数不带参数。构造函数的可见性将被公开。如果你不希望你的类
有一个公共构造函数，你需要声明与非缺省可见一个空的主构造函数：



``` kotlin
class DontCreateMe private constructor () {
}
```

> **NOTE**: On the JVM, if all of the parameters of the primary constructor have default values, the compiler will
> generate an additional parameterless constructor which will use the default values. This makes it easier to use
> Kotlin with libraries such as Jackson or JPA that create class instances through parameterless constructors.
在JVM上，如果所有的主构造函数的参数有默认值，编译器会产生一个额外的参数的构造函数，将使用默认值。
这使得更易于使用kotlin与通过参数构造函数创建类的实例，如Jackson或JPA库。


>
> ``` kotlin
> class Customer(val customerName: String = "")
> ```
{:.info}

### Creating instances of classes
创建类的实例

To create an instance of a class, we call the constructor as if it were a regular function:
要创建一个类的实例，我们调用构造函数，就好像它是一个常规的功能：

``` kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

Note that Kotlin does not have a *new*{: .keyword } keyword.
注意Kotlin不能有“new”关键字


### Class Members
类成员

Classes can contain
类可以包括

* Constructors and initializer blocks  构造和初始化模块
* [Functions](functions.html)
* [Properties](properties.html)
* [Nested and Inner Classes](nested-classes.html)
* [Object Declarations](object-declarations.html)


## Inheritance 继承

All classes in Kotlin have a common superclass `Any`, that is a default super for a class with no supertypes declared:
在Kotlin所有的类中都有一个共同的超类`Any`，这是一个默认的超级类且没有超类型声明：

``` kotlin
class Example // Implicitly inherits from Any
```

`Any` is not `java.lang.Object`; in particular, it does not have any members other than `equals()`, `hashCode()` and `toString()`.
Please consult the [Java interoperability](java-interop.html#object-methods) section for more details.
`Any`不属于`java.lang.Object`;特别是，它并没有任何其他任何成员，除了‘equals()','hashCode（）`和`toString（）`。
请参阅[Java的互操作性（java-interop.html#object-methods）更多的细节部分。


To declare an explicit supertype, we place the type after a colon in the class header:
要声明一个明确的父类，我们把类的头看作一个冒号后的：

``` kotlin
open class Base(p: Int)

class Derived(p: Int) : Base(p)
```

If the class has a primary constructor, the base type can (and must) be initialized right there,
using the parameters of the primary constructor.
如果类有一个主构造，基本类型可以（而且必须）来初始化就在那里，使用主构造的参数。

If the class has no primary constructor, then each secondary constructor has to initialize the base type
using the *super*{: .keyword } keyword, or to delegate to another constructor which does that.
Note that in this case different secondary constructors can call different constructors of the base type:
如果类没有主构造，那么每个次级构造函数初始化基本类型
使用*super*{：.keyword}关键字，或委托给另一个构造函数做到这一点。
注意，在这种情况下，不同的二级构造函数可以调用基类型的不同的构造：

``` kotlin
class MyView : View {
    constructor(ctx: Context) : super(ctx) {
    }

    constructor(ctx: Context, attrs: AttributeSet) : super(ctx, attrs) {
    }
}
```

The *open*{: .keyword } annotation on a class is the opposite of Java's *final*{: .keyword }: it allows others
to inherit from this class. By default, all classes in Kotlin are final, which
corresponds to [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html),
Item 17: *Design and document for inheritance or else prohibit it*.
在*open*{：.keyword}注释一类是Java的*最后*final*{：.keyword}：它允许他类
从这个类中继承。默认情况下，在Kotlin所有的类都是final，
对应于[Effective Java]（http://www.oracle.com/technetwork/java/effectivejava-136174.html）
项目17：*设计和文档继承，否则禁止其*。


### Overriding Members

As we mentioned before, we stick to making things explicit in Kotlin. And unlike Java, Kotlin requires explicit
annotations for overridable members (we call them *open*) and for overrides:
正如我们之前所提到的，我们坚持做的事情明确的Kotlin。不像Java中，Kotlin需要明确的
注释重写的成员（我们称之为*open*）和重写：

``` kotlin
open class Base {
  open fun v() {}
  fun nv() {}
}
class Derived() : Base() {
  override fun v() {}
}
```

The *override*{: .keyword } annotation is required for `Derived.v()`. If it were missing, the compiler would complain.
If there is no *open*{: .keyword } annotation on a function, like `Base.nv()`, declaring a method with the same signature in a subclass is illegal,
either with *override*{: .keyword } or without it. In a final class (e.g. a class with no *open*{: .keyword } annotation), open members are prohibited.
在*override*{：.keyword}注释需要`Derived.v（）`。如果没有，编译器会编译错误。
如果没有*open*{：.keyword}注释上的功能，比如`Base.nv（）'，声明的方法具有相同签名的一个子类是非法的，
无论是与*override*{：.keyword}或没有它。在最后的等级（例如一个类中，没有*open*{：.keyword}注释），开放式的成员被禁止。


A member marked *override*{: .keyword } is itself open, i.e. it may be overridden in subclasses. If you want to prohibit re-overriding, use *final*{: .keyword }:
成员标记为*override*{：.keyword}的本身是开放的，也就是说，它可以在子类中重写。如果你想禁止重写的，使用*final*{：.keyword}关键字：

``` kotlin
open class AnotherDerived() : Base() {
  final override fun v() {}
}
```

#### Wait! How will I hack my libraries now?!
等一下！我将如何利用我的库现在？

One issue with our approach to overriding (classes and members final by default) is that it would be difficult to subclass something inside the libraries you use to override some method that was not intended for overriding by the library designer, and introduce some nasty hack there.
其中一个问题是，我们的方法来重写（类和成员最终默认情况下）是，你用它来重写一些方法，利用子类的东西里面的库将会很困难，这并不适合由库设计覆盖库里面的，通过介绍一些讨厌的错误。

We think that this is not a disadvantage, for the following reasons:
我们认为这不是一个劣势，原因如下：

* Best practices say that you should not allow these hacks anyway
* People successfully use other languages (C++, C#) that have similar approach
* If people really want to hack, there still are ways: you can always write your hack in Java and call it from Kotlin (*see [Java Interop](java-interop.html)*), and Aspect frameworks always work for these purposes
最佳做法是，无论如何，你不应该允许这些破解的操作
*人们可以成功地使用其他语言（C ++，C＃），因为有类似的做法
*如果人们真的想破解，仍然有办法：你可以在Java中写你的操作，并从Kotlin调用它（*请参阅[Java的互操作（java的interop.html）*），这是Aspect框架一直努力的目标


### Overriding Rules 重写的规则

In Kotlin, implementation inheritance is regulated by the following rule: if a class inherits many implementations of the same member from its immediate superclasses,
it must override this member and provide its own implementation (perhaps, using one of the inherited ones).
To denote the supertype from which the inherited implementation is taken, we use *super*{: .keyword } qualified by the supertype name in angle brackets, e.g. `super<Base>`:
在Kotlin中，实现继承的调用通过以下规则：如果一个类继承超类成员的多种实现方法，可以直接在子类中引用，
它必须重写这个成员，并提供其自己的实现（也许，使用的继承者之一）。
为了表示从中继承的实现而采取的超类型，我们使用*super*{：.keyword}在尖括号，如规范的父名`super<Base>`：


``` kotlin
open class A {
  open fun f() { print("A") }
  fun a() { print("a") }
}

interface B {
  fun f() { print("B") } // interface members are 'open' by default
  fun b() { print("b") }
}

class C() : A(), B {
  // The compiler requires f() to be overridden:
  override fun f() {
    super<A>.f() // call to A.f()
    super<B>.f() // call to B.f()
  }
}
```

It's fine to inherit from both `A` and `B`, and we have no problems with `a()` and `b()` since `C` inherits only one implementation of each of these functions.
But for `f()` we have two implementations inherited by `C`, and thus we have to override `f()` in `C`
and provide our own implementation that eliminates the ambiguity.
它的优良来自`A`和`B`继承，我们没有问题，'a（）`和`B（）``,继承C`唯一一个实现所有这些功能。
但对于'F（）'，我们有两种实现方式继承了`C`，因此，我们必须重写`F（）`中的'C`
并提供自己的实现，从而消除了不确定性。



## Abstract Classes 抽象类

A class and some of its members may be declared *abstract*{: .keyword }.
An abstract member does not have an implementation in its class.
Thus, when some descendant inherits an abstract member, it does not count as an implementation:
一个类和它的一些成员可以声明为*abstract*{：.keyword}。
抽象成员没有在它的类的实现方法。
因此，当一些子类继承一个抽象的成员，它并不算是一个实现：

``` kotlin
abstract class A {
  abstract fun f()
}

interface B {
  open fun f() { print("B") }
}

class C() : A(), B {
  // We are not required to override f()
}
```

Note that we do not need to annotate an abstract class or function with open – it goes without saying.

We can override a non-abstract open member with an abstract one

需要注意的是，我们并不需要标注一个抽象类或者开放的函数 - 不言而喻。

我们可以重写一个开放与抽象的非抽象成员

``` kotlin
open class Base {
  open fun f() {}
}

abstract class Derived : Base() {
  override abstract fun f()
}
```

## Companion Objects  同伴对象

In Kotlin, unlike Java or C#, classes do not have static methods. In most cases, it's recommended to simply use
package-level functions instead.
在Kotlin中，不像Java或C＃，类没有静态方法。在大多数情况下，它建议简单地使用
包级函数。

If you need to write a function that can be called without having a class instance but needs access to the internals
of a class (for example, a factory method), you can write it as a member of an [object declaration](object-declarations.html)
inside that class.
如果你需要写一个可以调用的函数，而非一个类的实例，但需要访问的内部
一个类（例如，一个工厂方法），你可以写为[对象声明】（object_declarations.html）中的一员
里面的那个类。


Even more specifically, if you declare a [companion object](object-declarations.html#companion-objects) inside your class,
you'll be able to call its members with the same syntax as calling static methods in Java/C#, using only the class name
as a qualifier.
更具体地讲，如果声明一个[同伴对象（object-declarations.html#companion-objects）的类中，
你就可以在Java/ C＃中调用与它的成员方法相同的语法的静态方法，只使用类名
作为一个注解。
