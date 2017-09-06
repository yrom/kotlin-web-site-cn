---
type: doc
layout: reference
category: "Other"
title: "相等性"
---

# 相等性

Kotlin 中有两种类型的相等性：

* 引用相等（两个引用指向同一对象）
* 结构相等（用 `equals()` 检查）

## 引用相等

引用相等由 `===`（以及其否定形式 `!==`）操作判断。`a === b`
当且仅当 `a` 和 `b` 指向同一个对象时求值为 true。

## 结构相等

结构相等由 `==`（以及其否定形式 `!=`）操作判断。按照惯例，像 `a == b` 这样的表达式会翻译成

``` kotlin
a?.equals(b) ?: (b === null)
```

也就是说如果 `a` 不是 `null` 则调用 `equals(Any?)` 函数，否则（即 `a` 是 `null`）检查 b 是否与 `null` 引用相等。

请注意，当与 `null` 显式比较时完全没必要优化你的代码：`a == null` 会被自动转换为 `a=== null`。

## Floating point numbers equality

When an equality check operands are statically known to be `Float` or `Double` (nullable or not), the check follows the IEEE 754 
Standard for Floating-Point Arithmetic. 

Otherwise, the structural equality is used, which disagrees with the standard so that `NaN` is equal to itself, and `-0.0` is not equal to `0.0`.

See: [Floating Point Numbers Comparison](basic-types.html#floating-point-numbers-comparison)