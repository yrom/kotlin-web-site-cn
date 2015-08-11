---
type: doc
layout: reference
category: "Syntax"
title: "Returns and Jumps"
---

# Returns and Jumps
#返回和跳转


Kotlin has three structural jump operators

Kotlin 有三种跳出结构

* *return*{: .keyword }. By default returns from the nearest enclosing function or [function expression](lambdas.html#function-expressions).
* *return*{: .keyword }.默认情况下，从最近的一个封闭的方法或者 [方法表达式](lambdas.html#function-expressions)跳出.
* *break*{: .keyword }. Terminates the nearest enclosing loop.
* *break*{: .keyword }.终止最近的封闭循环
* *continue*{: .keyword }. Proceeds to the next step of the nearest enclosing loop.
* *continue*{: .keyword }.直接进入循环体的下次循环

# Break and Continue Labels
##中断和继续标签

Any expression in Kotlin may be marked with a *label*{: .keyword }.  
在Kotlin中任何表达式都可以用*label*{: .keyword }.来标记。  
Labels have the form of an identifier followed by the `@` sign, for example: `abc@`, `fooBar@` are valid labels (see the [grammar](grammar.html#label)).  
标签的格式是被'@'标识符标记，例如：`abc@`, `fooBar@`都是有效的标签（参见[语法](grammar.html#label)）  

To label an expression, we just put a label in front of it  
一个带标签的方法，我们只是放一个标签在他前面
``` kotlin
loop@ for (i in 1..100) {
  // ...
}
```

Now, we can qualify a *break*{: .keyword } or a *continue*{: .keyword } with a label:  
现在，我们可以将标签与 *break*{: .keyword } 或者*continue*{: .keyword }一起使用

``` kotlin
loop@ for (i in 1..100) {
  for (j in 1..100) {
    if (...)
      break@loop
  }
}
```

A *break*{: .keyword } qualified with a label jumps to the execution point right after the loop marked with that label.  
这个被loop标签标记的*break*{: .keyword }将跳出被loop标签标记的循环体

A *continue*{: .keyword } proceeds to the next iteration of that loop.  
*continue*{: .keyword }将进入循环体的下次循环


# Return at Labels
##返回标签

With function literals, local functions and object expression, functions can be nested in Kotlin.   
在Kotlin里，函数字面量、局部函数和对象表达式等函数都可以被嵌套在一起
Qualified *return*{: .keyword }s allow us to return from an outer function.   
适当的返回方式允许我们从外部方法返回值  

The most important use case is returning from a function literal. Recall that when we write this:  
最重要的使用方面是从一个函数字面量返回。当我们像下面这样写的时候，就会回调他

``` kotlin
fun foo() {
  ints.forEach {
    if (it == 0) return
    print(it)
  }
}
```

The *return*{: .keyword }-expression returns from the nearest enclosing function, i.e. `foo`.  
这个 *return*{: .keyword }表达式从最近的封闭的方法中返回，例如‘foo’。
(Note that such non-local returns are supported only for function literals passed to [inline-functions](inline-functions.html).)  
 (注意，非全局的返回只支持内部方法，参见[内联方法](inline-functions.html).)
If we need to return from a function literal, we have to label it and qualify the *return*{: .keyword }:  
如果我们只是需要跳出内部方法，我们必须标记它并且返回这个标签
``` kotlin
fun foo() {
  ints.forEach lit@ {
    if (it == 0) return@lit
    print(it)
  }
}
```

Now, it returns only from the function literal. Oftentimes it is more convenient to use implicits labels:  
现在只是从内部方法返回。有时候用匿名的标签将会更加方便 
such a label has the same name as the function to which the lambda is passed.  
像这样和方法同名的标签是可以的

``` kotlin
fun foo() {
  ints.forEach {
    if (it == 0) return@forEach
    print(it)
  }
}
```

Alternatively, we can replace the function literal with a [function expression](lambdas.html#function-expressions).
A *return*{: .keyword } statement in a function expression will return from the function expression itself.  
通常，我们用一个[方法表达式](lambdas.html#function-expressions)替代内部匿名方法。在方法内部声明一个*return*{: .keyword }将从其内部返回

``` kotlin
fun foo() {
  ints.forEach(fun(value: Int) {
    if (value == 0) return
    print(value)
  })
}
```

When returning a value, the parser gives preference to the qualified return, i.e.
当要返回一个值得时候，推荐使用描述性的返回，例如：
``` kotlin
return@a 1
```

means "return `1` at label `@a`" and not "return a labeled expression `(@a 1)`".  
意思是“返回被标记为‘@a’值是‘1’的标签，而不是像‘（@a 1）’的一个标签表达式”

Named functions automatically define labels:  
被命名的方法自动被定义成为标签

``` kotlin
fun outer() {
  fun inner() {
    return@outer // the label @outer was defined automatically
  }
}                                                                             
```
