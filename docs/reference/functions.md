---
type: doc
layout: reference
category: "Syntax"
title: "Functions"
---

# 函数

## 函数声明

在Kotlin中，函数声明使用关键字 *fun*{: .keyword }

``` kotlin
fun double(x: Int): Int {
}
```

## 函数用途

调用函数使用传统的方法

``` kotlin
val result = double(2)
```


调用成员函数使用点表达式

``` kotlin
Sample().foo() // create instance of class Sample and calls foo
```

### 中缀表示法 

函数还可以用中缀表示法，当

* 他们是成员函数 或者 [扩展函数](extensions.html)
* 他们只有一个参数
* They are marked with the `infix` keyword

``` kotlin
// Define extension to Int
infix fun Int.shl(x: Int): Int {
...
}

// call extension function using infix notation

1 shl 2

// is the same as

1.shl(2)
```

### 参数

函数参数是使用Pascal表达式，即 *name*: *type*。参数用逗号隔开。每个参数必须有显式类型。

``` kotlin
fun powerOf(number: Int, exponent: Int) {
...
}
```

### 默认参数(缺省参数)

函数参数有默认值,当对应的参数是省略。与其他语言相比可以减少数量的过载。

``` kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size()) {
...
}
```

默认值定义使用后* * = * *类型的值。

### 命名参数

可以在调用函数时使用命名的函数参数。当一个函数有大量的参数或默认参数时这非常方便。

给出下面的函数:

``` kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
...
}
```

我们可以使用默认参数来调用这个

``` kotlin
reformat(str)
```

然而，调用非默认时，调用类似于

``` kotlin
reformat(str, true, true, false, '_')
```

使用命名参数我们可以使代码更具有可读性

``` kotlin
reformat(str,
    normalizeCase = true,
    upperCaseFirstLetter = true,
    divideByCamelHumps = false,
    wordSeparator = '_'
  )
```

如果我们不需要所有的参数

``` kotlin
reformat(str, wordSeparator = '_')
```

Note that the named argument syntax cannot be used when calling Java functions, because Java bytecode does not
always preserve names of function parameters.


### 返回Unit的函数

如果一个函数不返回任何有用的值，它的返回类型是`Unit`。Unit`是一种只有一个值 - `Unit`。这个
值不需要显式地返回

``` kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` or `return` is optional
}
```

`Unit`返回类型声明也是可选的。上面的代码等同于

``` kotlin
fun printHello(name: String?) {
    ...
}
```

### 单个表达式函数

当一个函数返回单个表达式，花括号可以省略并且主体由** =**符号之后指定

``` kotlin
fun double(x: Int): Int = x * 2
```

显式地声明返回类型[可选](# explicit-return-types)时,这可以由编译器推断

``` kotlin
fun double(x: Int) = x * 2
```

### 显式地返回类型

函数模块体必须显式地指定返回类型，除非是用于返回`Unit`， [在这种情况下](#unit-returning-functions)，它是可选的。
Kotlin不推断返回类型与函数在模块体的功能，因为这些功能可能在模块体有复杂的控制流程，
对于阅读者（有时甚至编译器）来说返回类型将不明显。


### 数量可变的参数(可变参数)

函数的（通常最后一个）参数可以使用'vararg`修饰：

``` kotlin
fun <T> asList(vararg ts: T): List<T> {
  val result = ArrayList<T>()
  for (t in ts) // ts is an Array
    result.add(t)
  return result
}
```

允许可变参数传递给函数:

```kotlin
  val list = asList(1, 2, 3)
```

内部函数`vararg`类型`T`是可见的array`T`,即上面的例子中的`ts`变量是`Array<out T>`类型。

只有一个参数可以标注为 `vararg`.If a `vararg` parameter is not the last one in the list, values for the
following parameters can be passed using the named argument syntax, or, if the parameter has a function type, by passing
a lambda outside parentheses.

当我们调用`vararg`函数，我们可以一个接一个传递参数，例如 `asList(1, 2, 3)`或者，如果我们已经有了一个数组
并希望将其内容传递给函数，我们使用**spread** 操作符（在数组前面加`*`）：

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

## 函数作用域(函数范围)

在Kotlin中函数可以在文件顶级声明，这意味着您不需要像一些语言如Java、C#或Scala那样创建一个类来持有一个函数。此外
除了顶级函数功能，Kotlin函数也可以在局部声明，作为成员函数和扩展函数.

### 局部函数

Kotlin提供局部函数,即一个函数在另一个函数中

``` kotlin
fun dfs(graph: Graph) {
  fun dfs(current: Vertex, visited: Set<Vertex>) {
    if (!visited.add(current)) return
    for (v in current.neighbors)
      dfs(v, visited)
  }

  dfs(graph.vertices[0], HashSet())
}
```

局部函数可以访问外部函数的局部变量（即闭包），所以在上面的例子，the *visited*是局部变量.

``` kotlin
fun dfs(graph: Graph) {
  val visited = HashSet<Vertex>()
  fun dfs(current: Vertex) {
    if (!visited.add(current)) return
    for (v in current.neighbors)
      dfs(v)
  }

  dfs(graph.vertices[0])
}
```

### Member Functions
### 成员函数

成员函数是一个函数,定义在一个类或对象里

``` kotlin
class Sample() {
  fun foo() { print("Foo") }
}
```

成员函数调用点符号

``` kotlin
Sample().foo() // creates instance of class Sample and calls foo
```

有关类信息和主要成员查看[Classes](classes.html) 和 [Inheritance](classes.html#inheritance)

### 重载函数

函数可以有泛型参数，通过在函数前使用尖括号指定。

``` kotlin
fun <T> singletonList(item: T): List<T> {
  // ...
}
```

有关重载函数更多信息请查看 [Generics](generics.html)

### 内联函数

内联函数解释 [here](inline-functions.html)

### 扩展函数

扩展函数解释 [their own section](extensions.html)

### 高阶函数和Lambdas表达式

高阶函数和Lambdas表达式中有详细解释 [their own section](lambdas.html)

## Tail recursive functions

Kotlin supports a style of functional programming known as [tail recursion](https://en.wikipedia.org/wiki/Tail_call).
This allows some algorithms that would normally be written using loops to instead be written using a recursive function, but without the risk of stack overflow.
When a function is marked with the `tailrec` modifier and meets the required form the compiler optimises out the recursion, leaving behind a fast and efficient loop based version instead.

``` kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

This code calculates the fixpoint of cosine, which is a mathematical constant. It simply calls Math.cos repeatedly starting at 1.0 until the result doesn't change any more, yielding a result of 0.7390851332151607. The resulting code is equivalent to this more traditional style:

``` kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```

To be eligible for the `tailrec` modifier, a function must call itself as the last operation it performs. You cannot use tail recursion when there is more code after the recursive call, and you cannot use it within try/catch/finally blocks. Currently tail recursion is only supported in the JVM backend.

---

翻译By Jacky Xu

