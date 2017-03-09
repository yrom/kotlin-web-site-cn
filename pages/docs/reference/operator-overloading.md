---
type: doc
layout: reference
title: "操作符符重载"
category: "Syntax"
---

# 操作符符重载

Kotlin 允许我们为自己的类型提供预定义的一组操作符的实现。这些操作符具有固定的符号表示
（如 `+` 或 `*`）和固定的[优先级](grammar.html#precedence)。为实现这样的操作符，我们为相应的类型（即二元操作符左侧的类型和一元操作符的参数类型）提供了一个固定名字的[成员函数](functions.html#成员函数)
或[扩展函数](extensions.html)。
重载操作符的函数需要用 `operator` 修饰符标记。

## 约定

在这里我们描述为不同操作符规范操作符重载的约定。

### 一元操作

| 表达式 | 翻译为 |
|------------|---------------|
| `+a` | `a.unaryPlus()` |
| `-a` | `a.unaryMinus()` |
| `!a` | `a.not()` |

这个表是说，当编译器处理例如表达式 `+a` 时，它执行以下步骤：

* 确定 `a` 的类型，令其为 `T`。
* 为接收者 `T` 查找一个带有 `operator` 修饰符的无参函数 `unaryPlus（）`，即成员函数或扩展函数。
* 如果函数不存在或不明确，则导致编译错误。
* 如果函数存在且其返回类型为 `R`，那就表达式 `+a` 具有类型 `R`。

*注意* 这些操作以及所有其他操作都针对[基本类型](basic-types.html)做了优化，不会为它们引入函数调用的开销。

| 表达式 | 翻译为 |
|------------|---------------|
| `a++` | `a.inc()` + 见下文 |
| `a--` | `a.dec()` + 见下文 |

The `inc()` and `dec()` functions must return a value, which will be assigned to the variable on which the
`++` or `--` operation was used. They shouldn't mutate the object on which the `inc` or `dec` was invoked.

编译器执行以下步骤来解析*后缀*形式的操作符，例如 `a++`：

* 确定 `a` 的类型，令其为 `T`。
* 查找一个适用于类型为 `T` 的接收者的、带有 `operator` 修饰符的无参数函数 `inc()`。
* Checks that the return type of the function is a subtype of `T`.

计算表达式的步骤是：

* 把 `a` 的初始值存储到临时存储 `a0` 中，
* 把 `a.inc()` 结果赋值给 `a`，
* 把 `a0` 作为表达式的结果返回。

对于 `a--`，步骤是完全类似的。

对于*前缀*形式 `++a` 和 `--a` 以相同方式解析，其步骤是：

* 把 `a.inc()` 结果赋值给 `a`，
* 把 `a` 的新值作为表达式结果返回。

### 二元操作

| 表达式 | 翻译为 |
| -----------|-------------- |
| `a + b` | `a.plus(b)` |
| `a - b` | `a.minus(b)` |
| `a * b` | `a.times(b)` |
| `a / b` | `a.div(b)` |
| `a % b` | `a.rem(b)`, `a.mod(b)` (deprecated) |
| `a..b ` | `a.rangeTo(b)` |

对于此表中的操作，编译器只是解析成*翻译为*列中的表达式。

Note that the `rem` operator is supported since Kotlin 1.1. Kotlin 1.0 uses the `mod` operator, which is deprecated
in Kotlin 1.1.

| Expression | Translated to |
| -----------|-------------- |
| `a in b` | `b.contains(a)` |
| `a !in b` | `!b.contains(a)` |

对于 `in` 和 `!in`，过程是相同的，但是参数的顺序是相反的。
{:#in}

| 表达式 | 翻译为 |
| -------|-------------- |
| `a[i]`  | `a.get(i)` |
| `a[i, j]`  | `a.get(i, j)` |
| `a[i_1, ...,  i_n]`  | `a.get(i_1, ...,  i_n)` |
| `a[i] = b` | `a.set(i, b)` |
| `a[i, j] = b` | `a.set(i, j, b)` |
| `a[i_1, ...,  i_n] = b` | `a.set(i_1, ..., i_n, b)` |

方括号转换为调用带有适当数量参数的 `get` 和 `set`。

| 表达式 | 翻译为 |
|--------|---------------|
| `a()`  | `a.invoke()` |
| `a(i)`  | `a.invoke(i)` |
| `a(i, j)`  | `a.invoke(i, j)` |
| `a(i_1, ...,  i_n)`  | `a.invoke(i_1, ...,  i_n)` |

圆括号转换为调用带有适当数量参数的 `invoke`。

| 表达式 | 翻译为 |
|------------|---------------|
| `a += b` | `a.plusAssign(b)` |
| `a -= b` | `a.minusAssign(b)` |
| `a *= b` | `a.timesAssign(b)` |
| `a /= b` | `a.divAssign(b)` |
| `a %= b` | `a.modAssign(b)` |
{:#assignments}

对于赋值操作，例如 `a += b`，编译器执行以下步骤：

* 如果右列的函数可用
  * 如果相应的二元函数（即 `plusAssign()` 对应于 `plus()`）也可用，那么报告错误（模糊）。
  * 确保其返回类型是 `Unit`，否则报告错误。
  * 生成 `a.plusAssign(b)` 的代码
* 否则试着生成 `a = a + b` 的代码（这里包含类型检查：`a + b` 的类型必须是 `a` 的子类型）。

*注意*：赋值在 Kotlin 中*不是*表达式。
{:#Equals}

| 表达式 | 翻译为 |
|------------|---------------|
| `a == b` | `a?.equals(b) ?: (b === null)` |
| `a != b` | `!(a?.equals(b) ?: (b === null))` |

*注意*：`===` 和 `!==`（同一性检查）不可重载，因此不存在对他们的约定

这个 `==` 操作符有些特殊：它被翻译成一个复杂的表达式，用于筛选 `null` 值。
`null == null` is always true, and `x == null` for a non-null `x` is always false and won't invoke `x.equals()`.

| 表达式 | 翻译为 |
|--------|---------------|
| `a > b`  | `a.compareTo(b) > 0` |
| `a < b`  | `a.compareTo(b) < 0` |
| `a >= b` | `a.compareTo(b) >= 0` |
| `a <= b` | `a.compareTo(b) <= 0` |

所有的比较都转换为对 `compareTo` 的调用，这个函数需要返回 `Int` 值

## 命名函数的中缀调用

我们可以通过[中缀函数的调用](functions.html#中缀表示法) 来模拟自定义中缀操作符。
