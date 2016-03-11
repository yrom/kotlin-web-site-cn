---
type: doc
layout: reference
category: "Syntax"
title: "类型的检查与转换"
---

# 类型的检查与转换

## `is` 和 `!is`运算符


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

## 智能转换


在很多情况下，在Kotlin有时不用使用明确的转换运算符，因为编译器会在需要的时候自动为了不变的值和输入（安全）而使用`is`进行监测：

``` kotlin
fun demo(x: Any) {
  if (x is String) {
    print(x.length) // x is automatically cast to String
  }
}
```


如果错误的检查导致返回，编译器会清楚地转换为一个正确的：

``` kotlin
  if (x !is String) return
  print(x.length) // x is automatically cast to String
```


或者在右边是`&&` 和 `||`：

``` kotlin
  // x is automatically cast to string on the right-hand side of `||`
  if (x !is String || x.length == 0) return

  // x is automatically cast to string on the right-hand side of `&&`
  if (x is String && x.length > 0)
      print(x.length) // x is automatically cast to String
```



这些智能转换在 [*when*{: .keyword }-expressions](control-flow.html#when-expressions)
和 [*while*{: .keyword }-loops](control-flow.html#while-loops) 也一样：

``` kotlin
when (x) {
  is Int -> print(x + 1)
  is String -> print(x.length + 1)
  is IntArray -> print(x.sum())
}
```

Note that smart casts do not work when the compiler cannot guarantee that the variable cannot change between the check and the usage.
More specifically, smart casts are applicable according to the following rules:

  * *val*{: .keyword } local variables - always;
  * *val*{: .keyword } properties - if the property is private or internal or the check is performed in the same module where the property is declared. Smart casts aren't applicable to open properties or properties that have custom getters;
  * *var*{: .keyword } local variables - if the variable is not modified between the check and the usage and is not captured in a lambda that modifies it;
  * *var*{: .keyword } properties - never (because the variable can be modified at any time by other code).


## “不安全”的转换运算符

通常，如果转换是不可能的，转换运算符会抛出一个异常。于是，我们称之为*不安全的*。在Kotlin这种不安全的转换会出现在插入运算符*as*{: .keyword } (see [operator precedence](grammar.html#operator-precedence))：

``` kotlin
val x: String = y as String
```


记住*null*{: .keyword }不能被转换为[不可为空的](null-safety.html)`String`。例如，如果`y`是空，则这段代码会抛出异常。为了匹配Jave的转换语义，我们不得不在右边拥有可空的类型，就像：

``` kotlin
val x: String? = y as String?
```

## “安全的”（可为空的）转换运算符


为了避免异常的抛出，一个可以使用*安全的*转换运算符——*as?*{: .keyword } ，它可以在失败时返回一个*null*{: .keyword }：

``` kotlin
val x: String? = y as? String
```


记住尽管事实是右边的*as?*{: .keyword }可使一个不为空的`String`类型的转换结果为可空的。





