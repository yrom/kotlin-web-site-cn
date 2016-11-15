---
type: doc
layout: reference
category: "Syntax"
title: "嵌套类"
---

# 嵌套类

在类的内部可以嵌套其他的类

``` kotlin
class Outer {
    private val bar: Int = 1
    class Nested {
        fun foo() = 2
    }
}

val demo = Outer.Nested().foo() // == 2
```

## 内部类

为了能被外部类访问一个类可以被标记为内部类（“inner” 关键词）。内部类会带有一个来自外部类的对象的引用：  

``` kotlin
class Outer {
    private val bar: Int = 1
    inner class Inner {
        fun foo() = bar
    }
}

val demo = Outer().Inner().foo() // == 1
```

参见[this-expressions.html](this-expressions.html)中“this”关键词用法来学习在内部类中如何消除“this”关键词的歧义。

## Anonymous inner classes

Anonymous inner class instances are created using an [object expression](object-declarations.html#object-expressions):
                                                      
``` kotlin
window.addMouseListener(object: MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) {
        // ...
    }
                                                                                                            
    override fun mouseEntered(e: MouseEvent) {
        // ...
    }
})
```

If the object is an instance of a functional Java interface (i.e. a Java interface with a single abstract method),
you can create it using a lambda expression prefixed with the type of the interface:

``` kotlin
val listener = ActionListener { println("clicked") }
```
