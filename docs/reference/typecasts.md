---
type: doc
layout: reference
category: "Syntax"
title: "Type Checks and Casts"
---

# Type Checks and Casts 类型的检查与转换

## `is` and `!is` Operators `is` 和 `!is`运算符

We can check whether an object conforms to a given type at runtime by using the `is` operator or its negated form `!is`:

我们可以使用`is` 或者它的否定`!is`运算符检查一个对象在运行中是否符合所给出的类型：

``` kotlin
if (obj is String) {
  print(obj.length)
}

if (obj !is String) { // same as !(obj is String)
  print("Not a String")
}
else {
  print(obj.length)
}
```

## Smart Casts 智能转换

In many cases, one does not need to use explicit cast operators in Kotlin, because the compiler tracks the
`is`-checks for immutable values and inserts (safe) casts automatically when needed:

在很多情况下，在Kotlin有时不用使用明确的转换运算符，因为编译器会在需要的时候自动为了不变的值和输入（安全）而使用`is`进行监测：

``` kotlin
fun demo(x: Any) {
  if (x is String) {
    print(x.length) // x is automatically cast to String
  }
}
```

The compiler is smart enough to know a cast to be safe if a negative check leads to a return:

如果错误的检查导致返回，编译器会清楚地转换为一个正确的：

``` kotlin
  if (x !is String) return
  print(x.length) // x is automatically cast to String
```

or in the right-hand side of `&&` and `||`:

或者在右边是`&&` 和 `||`：

``` kotlin
  // x is automatically cast to string on the right-hand side of `||`
  if (x !is String || x.length == 0) return

  // x is automatically cast to string on the right-hand side of `&&`
  if (x is String && x.length > 0)
      print(x.length) // x is automatically cast to String
```


Such _smart casts_ work for [*when*{: .keyword }-expressions](control-flow.html#when-expressions)
and [*while*{: .keyword }-loops](control-flow.html#while-loops) as well:

这些智能转换在 [*when*{: .keyword }-expressions](control-flow.html#when-expressions)
和 [*while*{: .keyword }-loops](control-flow.html#while-loops) 也一样：

``` kotlin
when (x) {
  is Int -> print(x + 1)
  is String -> print(x.length + 1)
  is Array<Int> -> print(x.sum())
}
```


## "Unsafe" cast operator “不安全”的转换运算符

Usually, the cast operator throws an exception if the cast is not possible. Thus, we call it *unsafe*.
The unsafe cast in Kotlin is done by the infix operator *as*{: .keyword } (see [operator precedence](grammar.html#operator-precedence)):

通常，如果转换是不可能的，转换运算符会抛出一个异常。于是，我们称之为*不安全的*。在Kotlin这种不安全的转换会出现在插入运算符*as*{: .keyword } (see [operator precedence](grammar.html#operator-precedence))：

``` kotlin
val x: String = y as String
```

Note that *null*{: .keyword } cannot be cast to `String` as this type is not [nullable](null-safety.html),
i.e. if `y` is null, the code above throws an exception.
In order to match Java cast semantics we have to have nullable type at cast right hand side, like

记住*null*{: .keyword }不能被转换为[不可为空的](null-safety.html)`String`。例如，如果`y`是空，则这段代码会抛出异常。为了匹配Jave的转换语义，我们不得不在右边拥有可空的类型，就像：

``` kotlin
val x: String? = y as String?
```

## "Safe" (nullable) cast operator “安全的”（可为空的）转换运算符

To avoid an exception being thrown, one can use a *safe* cast operator *as?*{: .keyword } that returns *null*{: .keyword } on failure:

为了避免异常的抛出，一个可以使用*安全的*转换运算符——*as?*{: .keyword } ，它可以在失败时返回一个*null*{: .keyword }： 

``` kotlin
val x: String? = y as? String
```

Note that despite the fact that the right-hand side of *as?*{: .keyword } is a non-null type `String` the result of the cast is nullable.

记住尽管事实是右边的*as?*{: .keyword }可使一个不为空的`String`类型的转换结果为可空的。


----

翻译By [Wahchi](https://github.com/wahchi)


