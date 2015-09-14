---
type: doc
layout: reference
title: "Operator overloading"
category: "Syntax"
---

# 运算符重载

Kotlin允许我们实现一些我们自定义类型的运算符实现。这些运算符有固定的表示(像 `+` 或者 `*`) ，和固定的[优先级](grammar.html#precedence)。为实现这样的运算符，我们提供了固定名字的[成员函数](functions.html#member-functions)和[扩展函数](extensions.html)，比如二元运算符的左值和一元运算符的参数类型。 

## 转换

这里我们描述了一些常用运算符的重载。

### 一元运算符

| 表达式 | 翻译为 |
|------------|---------------|
| `+a` | `a.plus()` |
| `-a` | `a.minus()` |
| `!a` | `a.not()` |

这张表解释了当编译器运行时，比如，表达式 `+a` ，是这样运行的：

* 决定`a`的类型, 假设为`T`。
* 寻找接收者是`T`的无参函数`plus()`,例如一个成员方法或者扩展方法。
* 如果找不到或者不明确就返回一个错误。
* 如果函数是当前函数或返回类型是`R`则表达式`+a`是`R`类型。

*注意* 这些操作符和其它的一样, 都被优化为[基本类型](basic-types.html)并且不会产生多余的开销。

| 表达式 | 翻译为 |
|------------|---------------|
| `a++` | `a.inc()` + see below |
| `a--` | `a.dec()` + see below |


这些操作符允许修改接收者和返回类型。

> **`inc()/dec()` shouldn't mutate the receiver object**.<br>
> By "changing the receiver" we mean _the receiver-variable_, not the receiver object.
{:.note}

编译器是这样解决有*后缀*的操作符的比如`a++`:

* 决定`a`的类型, 假设为`T`。
* Looks up a function `inc()` with no parameters, applicable to the receiver of type `T`.
* If the function returns a type `R`, then it must be a subtype of `T`.

计算表达式的效果是：

* Store the initial value of `a` to a temporary storage `a0`,
* Assign the result of `a.inc()` to `a`,
* Return `a0` as a result of the expression.

a-- 的运算步骤也是一样的。

对于前缀运算符`++a`和`--a`的解决方式也是一样的, 效果是:

* Assign the result of `a.inc()` to `a`,
* Return the new value of `a` as a result of the expression.

### 二元操作符

| 表达式 | 翻译为 |
| -----------|-------------- |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.mod(b)` |
| `a..b ` | `a.rangeTo(b)` |

编译器只是解决了该表中翻译为列的表达式。

| Expression | Translated to |
| -----------|-------------- |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |

in 和 !in 的产生步骤是一样的，但参数顺序是相反的。
{:#in}

| 标志 | 翻译为 |
| -------|-------------- |
| `a[i]`  | `a.get(i)` |
| `a[i, j]`  | `a.get(i, j)` |
| `a[i_1, ...,  i_n]`  | `a.get(i_1, ...,  i_n)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a[i_1, ...,  i_n] = b` | `a.set(i_1, ..., i_n, b)` |

方括号被转换为 get set 函数。

| 标志 | 翻译为 |
|--------|---------------|
| `a(i)`  | `a.invoke(i)` |
| `a(i, j)`  | `a.invoke(i, j)` |
| `a(i_1, ...,  i_n)`  | `a.invoke(i_1, ...,  i_n)` |

括号被转换为带有正确参数的 invoke 函数。

| 表达式 | 翻译为 |
|------------|---------------|
| `a += b` | `a.plusAssign(b)` |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)` |
| `a %= b` | `a.modAssign(b)` |
{:#assignments}

在分配 a+= b时编译器是下面这样实现的:

* If the function from the right column is available
  * If the corresponding binary function (i.e. `plus()` for `plusAssign()`) is available too, report error (ambiguity).
  * Make sure its return type is `Unit`, and report an error otherwise.
  * Generate code for `a.plusAssign(b)`
* Otherwise, try to generate code for `a = a + b` (this includes a type check: the type of `a + b` must be a subtype of `a`).

*注意*: assignments在Kotlin中不是表达式.
{:#Equals}

| 表达式 | 翻译为 |
|------------|---------------|
| `a == b` | `a?.equals(b) ?: b.identityEquals(null)` |
| `a != b` | `!(a?.equals(b) ?: b.identityEquals(null))` |

*注意*: `===` 和 `!==` (实例检查)不能重载, 所以没有转换方式。

`==`运算符有两点不同:

* 它被翻译成一个复杂的表达式，用于筛选空值，而且 `null == null` 是真.
* 它需要带有特定签名的函数，而不仅仅是特定名称的函数，像下面这样：

``` kotlin
fun equals(other: Any?): Boolean
```

或者用相同的参数列表和返回类型的扩展函数.

| 标志 | 翻译为 |
|--------|---------------|
| `a > b`  | `a.compareTo(b) > 0` |
| `a < b`  | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

所有的比较都转换为`compareTo`的调用，这个函数需要返回`Int`值

## 命名函数的中缀调用

我们可以通过[中缀函数的调用](functions.html#infix-notation).
来模拟自定义中缀操作符。