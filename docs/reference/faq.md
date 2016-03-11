---
type: doc
layout: reference
category: FAQ
title: FAQ
---

# FAQ

## 常见问题

### Kotlin是什么？

Kotlin 是目标平台为 JVM 和 JavaScript 的静态类型语言。它是一种旨在工业级使用的通用语言。

它是由 JetBrains 一个团队开发的，然而它是开源（OSS）语言并且也有外部贡献者。

### 为什么要出一门新语言？

在 JetBrains 我们已经在 Java 平台开发很长时间，并且我们知道它（Java）有多好。
另一方面，我们知道由于向后兼容性问题 Java 编程语言有一定的局限性和问题是不可能
或者很难解决的。我们知道 Java 还会延续很长时间，
但我们相信社区会从这个新的静态类型 JVM 平台语言中受益，它没有
遗留问题而有开发人员迫切想要特性。


这个项目背后的主要设计目标是：

* 创建一个兼容 Java 的语言，
* 它的编译速度至少和 Java 一样快，
* 使它比 Java 更安全，即静态检测空指针解引用等常见陷阱，
* 通过支持变量类型推断、高阶函数（闭包）、扩展函数、mixin 以及一等公民的委托等等使它比 Java 更简洁；
* 并且，在保持有用级表现力（见上文）的前提下，使它比最成熟的竞品——Scala 更简单。

### 如何授权？

Kotlin 是一种开源语言并在 Apache 2 开源软件许可下授权。它的 IntelliJ 插件也是开源软件。

它托管在 Github 上并且我们很乐意接受贡献者。


### 它兼容Java？

兼容。编译器生成的是 Java 字节码。Kotlin 可以调用 Java 并且 Java 也可以调用 Kotlin。参见[与 Java 互通性](java-interop.html)。

### 运行Kotlin代码所需的最低Java版本是哪个？

Kotlin 生成的字节码兼容 Java 6 以及更新版本。这确保 Kotlin 可以在像 Android 这样上一个所支持版本是 Java 6 的环境中使用。

### 有没有工具支持？

有。有一个作为 Apache 2 许可下开源项目的 IntelliJ IDEA 插件可用。 在
[自由开源社区版和旗舰版](http://www.jetbrains.com/idea/features/editions_comparison_matrix.html)的 IntelliJ IDEA 中都可以使用 Kotlin。

### 有没有Eclipse支持？

有。安装说明请参阅这个[教程](/docs/tutorials/getting-started-eclipse.html)。

### 有独立的编译器吗？

有。你可以从 [Github 上的发布页]({{site.data.releases.latest.url}})下载独立的编译器和其他构建工具。

### Kotlin是函数式语言吗？

Kotlin 是一种面向对象语言。不过它支持高阶函数以及 lambda 表达式和顶层函数。此外，在
Kotlin 标准库中还有很多一般函数式语言的设计（例如 map、flatMap、reduce 等）。当然，什么是**函数式语言**没有明确的定义，所以我们不能说 Kotlin 是其中之一。

### Kotlin支持泛型吗？

Kotlin 支持泛型。它也支持声明处型变和使用处型变。Kotlin 也没有通配符类型。内联函数支持具体化的类型参数。

### 分号是必需的吗？

不是。它们是可选的。

### 花括号是必需的吗？

是。

### 为什么类型声明在右侧？

我们相信这会使代码更易读。此外它启用了一些很好的语法特性，例如，很容易脱离类型注解。Scala 也已
很好地证明了这没有问题。

### 右侧类型声明会影响工具吗？

不会。我们仍然可以实现对变量名的建议等等。

### Kotlin是可扩展的吗？

我们计划使其在这几个方面可扩展：从内联函数到注解和类型加载器。

### 我可以把我的DSL嵌入到语言里吗？

可以。Kotlin 提供了一些有助于此的特性：操作符重载、通过内联函数自定义控制结构、中缀函数调用、扩展函数、注解以及
语言引文（language quotations）。

### JavaScript支持到ECMAScript的什么水平？

目前到 5。

### JavaScript后端支持模块系统吗？

支持。有提供 CommonJS 和 AMD 支持的计划。


