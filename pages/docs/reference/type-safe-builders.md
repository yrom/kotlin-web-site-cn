---
type: doc
layout: reference
category: "Syntax"
title: "类型安全的 Groovy-风格构建器"
---

# 类型安全的构建器

[构建器（builder）](http://www.groovy-lang.org/dsls.html#_nodebuilder)的概念在 *Groovy* 社区中非常热门。
构建器允许以半声明（semi-declarative）的方式定义数据。构建器很适合用来[生成 XML](http://www.groovy-lang.org/processing-xml.html#_creating_xml)、
[布局 UI 组件](http://www.groovy-lang.org/swing.html)、
[描述 3D 场景](http://www.artima.com/weblogs/viewpost.jsp?thread=296081)以及其他更多功能……

对于很多情况下，Kotlin 允许*检查类型*的构建器，这使得它们比
Groovy 自身的动态类型实现更具吸引力。

对于其余的情况，Kotlin 支持动态类型构建器。

## 一个类型安全的构建器示例

考虑下面的代码：

``` kotlin
import com.example.html.* // 参见下文声明

fun result(args: Array<String>) =
    html {
        head {
            title {+"XML encoding with Kotlin"}
        }
        body {
            h1 {+"XML encoding with Kotlin"}
            p  {+"this format can be used as an alternative markup to XML"}

            // 一个具有属性和文本内容的元素
            a(href = "http://kotlinlang.org") {+"Kotlin"}

            // 混合的内容
            p {
                +"This is some"
                b {+"mixed"}
                +"text. For more see the"
                a(href = "http://kotlinlang.org") {+"Kotlin"}
                +"project"
            }
            p {+"some text"}

            // 以下代码生成的内容
            p {
                for (arg in args)
                    +arg
            }
        }
    }
```

这是完全合法的 Kotlin 代码。
你可以[在这里](http://try.kotlinlang.org/#/Examples/Longer examples/HTML Builder/HTML Builder.kt)在线运行上文代码（修改它并在浏览器中运行）。

## 实现原理

让我们来看看 Kotlin 中实现类型安全构建器的机制。
首先，我们需要定义我们想要构建的模型，在本例中我们需要建模 HTML 标签。
用一些类就可以轻易完成。
例如，`HTML` 是一个描述 `<html>` 标签的类，也就是说它定义了像 `<head>` 和 `<body>` 这样的子标签。
（参见[下文](#declarations)它的声明。）

现在，让我们回想下为什么我们可以在代码中这样写：

``` kotlin
html {
 // ……
}
```

`html` 实际上是一个函数调用，它接受一个 [lambda 表达式](lambdas.html) 作为参数。
该函数定义如下：

``` kotlin
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```

这个函数接受一个名为 `init` 的参数，该参数本身就是一个函数。
该函数的类型是 `HTML.() -> Unit`，它是一个 _带接收者的函数类型_ 。
这意味着我们需要向函数传递一个 HTML 类型的实例（ _接收者_ ），
并且我们可以在函数内部调用该实例的成员。
该接收者可以通过 *this*{: .keyword } 关键字访问：

``` kotlin
html {
    this.head { /* …… */ }
    this.body { /* …… */ }
}
```

（`head` 和 `body` 是 `HTML` 的成员函数。）

现在，像往常一样，*this*{: .keyword } 可以省略掉了，我们得到的东西看起来已经非常像一个构建器了：

``` kotlin
html {
    head { /* …… */ }
    body { /* …… */ }
}
```

那么，这个调用做什么？ 让我们看看上面定义的 `html` 函数的主体。
它创建了一个 `HTML` 的新实例，然后通过调用作为参数传入的函数来初始化它
（在我们的示例中，归结为在HTML实例上调用 `head` 和 `body`），然后返回此实例。
这正是构建器所应做的。

`HTML` 类中的 `head` 和 `body` 函数的定义与 `html` 类似。
唯一的区别是，它们将构建的实例添加到包含 `HTML` 实例的 `children` 集合中：

``` kotlin
fun head(init: Head.() -> Unit) : Head {
    val head = Head()
    head.init()
    children.add(head)
    return head
}

fun body(init: Body.() -> Unit) : Body {
    val body = Body()
    body.init()
    children.add(body)
    return body
}
```

实际上这两个函数做同样的事情，所以我们可以有一个泛型版本，`initTag`：

``` kotlin
    protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }
```

所以，现在我们的函数很简单：

``` kotlin
fun head(init: Head.() -> Unit) = initTag(Head(), init)

fun body(init: Body.() -> Unit) = initTag(Body(), init)
```

并且我们可以使用它们来构建 `<head>` 和 `<body>` 标签。


这里要讨论的另一件事是如何向标签体中添加文本。在上例中我们这样写到

``` kotlin
html {
    head {
        title {+"XML encoding with Kotlin"}
    }
    // ……
}
```

所以基本上，我们只是把一个字符串放进一个标签体内部，但在它前面有一个小的 `+`，
所以它是一个函数调用，调用一个前缀 `unaryPlus()` 操作。
该操作实际上是由一个扩展函数 `unaryPlus()` 定义的，该函数是 `TagWithText` 抽象类（`Title` 的父类）的成员：

``` kotlin
fun String.unaryPlus() {
    children.add(TextElement(this))
}
```

所以，在这里前缀 `+` 所做的事情是把一个字符串包装到一个 `TextElement` 实例中，并将其添加到 `children` 集合中，
以使其成为标签树的一个适当的部分。

所有这些都在上面构建器示例顶部导入的包 `com.example.html` 中定义。
在下一节中，你可以阅读这个包的完整定义。

## `com.example.html` 包的完整定义

这就是 `com.example.html` 包的定义（只有上面例子中使用的元素）。
它构建一个 HTML 树。代码中大量使用了[扩展函数](extensions.html)和
[带接收者的 lambda 表达式](lambdas.html#带接收者的函数字面值)。

<a name='declarations'></a>

``` kotlin
package com.example.html

interface Element {
    fun render(builder: StringBuilder, indent: String)
}

class TextElement(val text: String) : Element {
    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent$text\n")
    }
}

abstract class Tag(val name: String) : Element {
    val children = arrayListOf<Element>()
    val attributes = hashMapOf<String, String>()

    protected fun <T : Element> initTag(tag: T, init: T.() -> Unit): T {
        tag.init()
        children.add(tag)
        return tag
    }

    override fun render(builder: StringBuilder, indent: String) {
        builder.append("$indent<$name${renderAttributes()}>\n")
        for (c in children) {
            c.render(builder, indent + "  ")
        }
        builder.append("$indent</$name>\n")
    }

    private fun renderAttributes(): String {
        val builder = StringBuilder()
        for (a in attributes.keys) {
            builder.append(" $a=\"${attributes[a]}\"")
        }
        return builder.toString()
    }


    override fun toString(): String {
        val builder = StringBuilder()
        render(builder, "")
        return builder.toString()
    }
}

abstract class TagWithText(name: String) : Tag(name) {
    operator fun String.unaryPlus() {
        children.add(TextElement(this))
    }
}

class HTML() : TagWithText("html") {
    fun head(init: Head.() -> Unit) = initTag(Head(), init)

    fun body(init: Body.() -> Unit) = initTag(Body(), init)
}

class Head() : TagWithText("head") {
    fun title(init: Title.() -> Unit) = initTag(Title(), init)
}

class Title() : TagWithText("title")

abstract class BodyTag(name: String) : TagWithText(name) {
    fun b(init: B.() -> Unit) = initTag(B(), init)
    fun p(init: P.() -> Unit) = initTag(P(), init)
    fun h1(init: H1.() -> Unit) = initTag(H1(), init)
    fun a(href: String, init: A.() -> Unit) {
        val a = initTag(A(), init)
        a.href = href
    }
}

class Body() : BodyTag("body")
class B() : BodyTag("b")
class P() : BodyTag("p")
class H1() : BodyTag("h1")

class A() : BodyTag("a") {
    public var href: String
        get() = attributes["href"]!!
        set(value) {
            attributes["href"] = value
        }
}

fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()
    html.init()
    return html
}
```
