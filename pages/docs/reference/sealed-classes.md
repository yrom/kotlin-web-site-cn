---
type: doc
layout: reference
category: "Classes and Objects"
title: "密封类"
---

# 密封类

密封类用来表示受限的类继承结构：当一个值为有限集中的
类型、而不能有任何其他类型时。在某种意义上，他们是枚举类的扩展：枚举类型的值集合
也是受限的，但每个枚举常量只存在一个实例，而密封类
的一个子类可以有可包含状态的多个实例。

要声明一个密封类，需要在类名前面添加 `sealed` 修饰符。虽然密封类也可以
有子类，但是所有子类声明都必须嵌套在这个密封类声明内部。

``` kotlin
sealed class Expr {
    class Const(val number: Double) : Expr()
    class Sum(val e1: Expr, val e2: Expr) : Expr()
    object NotANumber : Expr()
}
```

值得注意的是一个密封类的子类的继承者（间接继承）可以在任何地方声明，不一定要在
这个密封类声明内部。

使用密封类的关键好处在于使用 [`when` 表达式](control-flow.html#when-表达式) 的时候，如果能够
验证语句覆盖了所有情况，就不需要为该语句再添加一个 `else` 子句了。

``` kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Expr.Const -> expr.number
    is Expr.Sum -> eval(expr.e1) + eval(expr.e2)
    Expr.NotANumber -> Double.NaN
    // 不再需要 `else` 子句，因为我们已经覆盖了所有的情况
}
```

## 密封类的放宽规则（自 1.1 起）

### 同一文件中的子类

从 1.1 开始，你可以在顶层声明 `sealed` 类的子类，唯一的限制是它们应该和父类在同一个文件中。

### 密封类与数据类

数据类可以扩展其他类，包括 `sealed` 类，这使得继承结构更有用。

使用所有新支持的功能，你可以通过以下方式重写 `Expr` 类的继承结构：

``` kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()

fun eval(expr: Expr): Double = when (expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
}
```