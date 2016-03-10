---
type: doc
layout: reference
category: "Tools"
title: "生成kotlin代码文档"
---

# 生成kotlin代码文档

**KDoc**用来编写Kotlin代码文档（类似于java的 JavaDoc工具）。本质上来说，KDoc
结合了JavaDoc的标签块的句法和Markdown的语法来标记（来扩展Kotlin的特殊标记）。

## Generating the Documentation

Kotlin's documentation generation tool is called [Dokka](https://github.com/Kotlin/dokka). See the
[Dokka README](https://github.com/Kotlin/dokka/blob/master/README.md) for usage instructions.

Dokka has plugins for Gradle, Maven and Ant, so you can integrate documentation generation into your build process.

## KDoc 语法

像JavaDoc一样，KDoc注释也`/**`开头和也`*/`结束,每一行注释可能都是也星号开头的，
但是并不作为注释内容的一部分。

按惯例来说，文档的第一段（到第一行空白行结束）是该文档元素的
总体描述，接下来的注释是详细描述

每一个块标记也新一行开始并且也`@`字符开头

这是用 KDoc 写类文档的一个例子：

``` kotlin
/**
 * A group of *members*.
 *
 * This class has no useful logic; it's just a documentation example.
 *
 * @param T the type of a member in this group.
 * @property name the name of this group.
 * @constructor Creates an empty group.
 */
class Group<T>(val name: String) {
    /**
     * Adds a [member] to this group.
     * @return the new size of the group.
     */
    fun add(member: T): Int { ... }
}
```

## 块标签

KDoc现在支持如下的块标签：

#### `@param <name>`

代表一个函数的参数值或者一个类的参数。
为了更好的区分描述中的参数值，如果你喜欢，你可以在参数名
括在方括号中，下面是两个符合条件的句法：

```
@param name description.
@param[name] description.
```

#### `@return`

函数的返回值

#### `@constructor`

类构造函数

#### `@property <name>`

Documents the property of a class which has the specified name. This tag can be used for documenting properties
declared in the primary constructor, where putting a doc comment directly before the property definition would be
awkward.

#### `@throws <class>`, `@exception <class>`

用来标记一个方法抛出的异常。鉴于Kotlin没有异常检查，
因此不能期待所有可能异常都写出来，但是我们仍然可以使用这个标记来提示给这个类使用这一个
很好的信息。

#### `@sample <identifier>`

给当前的元素嵌入一个包含特殊名字的方法，为了能够包含例子
来展示这个元素是如何使用的。

#### `@see <identifier>`

给类或者方法加一个链接来**查看** 文档的信息

#### `@author`

文档编写人员的名字

#### `@since`

来指定什么版本引入了这个方法类

#### `@suppress`

不包含生成的文档中的元素。可用于不属于官方API的
模块的应用接口，但仍必须对外部可见。

> KDoc 不支持 `@deprecated` 这个标记. 请使用` @Deprecated`注释
{:.note}


## 内置Markup语法

内置Markup语法，KDoc使用了标准的[Markdown](http://daringfireball.net/projects/markdown/syntax) 语法,来扩展了
它支持在代码中链接到其他元素的速记语法。

### 链接到元素

为了链接到其它元素（类，方法，属性和参数），把它的元素放在中括号中：

```
Use the method [foo] for this purpose.
```

If you want to specify a custom label for the link, use the Markdown reference-style syntax:

```
Use [this method][foo] for this purpose.
```

您还可以在链接中使用限定名。需要注意的是，不同于javadoc，合格的名字总是使用点字符
分开的组件，即使在一个方法前：

```
Use [kotlin.reflect.KClass.properties] to enumerate the properties of the class.
```

如果被使用的元素内的元素被记录，则在链接的名称解析使用相同的规则。
特别是，这意味着，如果您已经导入一个名字到当前文件，在使用KDoc中您不需要完全限定它

注意KDoc在链接中没有解决重载成员的任何语法。自从Kotlin文档生成
工具把上所有的重载函数放在同一个页面之后，标识一个特定的重载函数
不需要链接的方式。
