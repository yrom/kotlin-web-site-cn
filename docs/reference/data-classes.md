---
type: doc
layout: reference
category: "Classes and Objects"
title: "Data Classes"
---

# 数据类

We frequently create classes that do nothing but hold data. In such classes some functionality is often mechanically
derivable from the data they hold. In Kotlin a class can be annotated as `data`:

我们经常创建一些只是处理数据的类。在这些类里的功能经常是衍生自他们所持有的数据。在Kotlin中，这样的类可以被称为`data`：
 
``` kotlin
data class User(val name: String, val age: Int)
```

This is called a _data class_. The compiler automatically derives the following members from all properties _declared in 
the primary constructor_:

这被叫做一个 _数据类_。编译器自动从在主构造函数定义的全部特性中得到以下成员：
  
  * `equals()`/`hashCode()` pair, 
  * `toString()` of the form `"User(name=John, age=42)"`,
  * [`componentN()` functions](multi-declarations.html) corresponding to the properties in their order or declaration,
  * `copy()` function (see below).


  
If any of these functions is explicitly defined in the class body or inherited from the base types, it will not be generated. 

如果有任一一个函数被明确地定义在类里或者被继承自基本类型，这将不会被生成。
  
*NOTE* that if a constructor parameter does not have a `val` or `var` in front of it, it will not be included in computation
of all these functions; nor will be properties declared in the class body or inherited from the superclass.

*NOTE*如果一个构造参数没有`val`或者`var`在前面，它将不会被包括进这些成员里;也不会在类里声明成属性或者继承自父类

> On the JVM, if the generated class needs to have a parameterless constructor, default values for all properties have to be specified
> (see [Constructors](classes.html#constructors)).
> 
> 在JVM中，如果生成的类需要含有一个无参的构造函数，则默认所有的属性已经被初始化了。
>
> ``` kotlin
> data class User(val name: String = "", val age: Int = 0)
> ```

## Copying复制
 

It's often the case that we need to copy an object altering _some_ of its properties, but keeping the rest unchanged. 
This is what `copy()` function is generated for. For the `User` class above, its implementation would be as follows:

在很多情况下，我们需要复制一个可以改变它里面_一些_属性但又保持剩下的属性不变的对象。这就是`copy()`这个方法的来源。对于上文的`User`类，应该是这么实现这个方法的
     
``` kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)     
```     

This allows us to write

也可以这么写

``` kotlin
val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

## Data Classes and Multi-Declarations数据类和多声明

_Component functions_ generated for data classes enable their use in [multi-declarations](multi-declarations.html):

_成员方法_用于使数据类可以[多声明]：

``` kotlin
val jane = User("Jane", 35) 
val (name, age) = jane
println("$name, $age years of age") // prints "Jane, 35 years of age"
```

## Standard Data Classes标准数据类

The standard library provides `Pair` and `Triple`. In most cases, though, named data classes are a better design choice, 
because they make the code more readable by providing meaningful names for properties.

在标准库提供了`Pair`和`Triple`。在很多情况下，即使命名数据类是一个更好的设计选择，因为这能让代码可读性更强