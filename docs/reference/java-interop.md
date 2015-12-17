---
type: doc
layout: reference
category: "Interop"
title: "Java Interop"
---

# Java交互

Kotlin 在设计时就是以与 java 交互为中心的。现存的 Java 代码可以在 kotlin 中使用，并且 Kotlin 代码也可以在 Java 中流畅运行。这节我们会讨论在 kotlin 中调用 Java 代码的细节。

## 在Kotlin中调用Java代码

基本所有的 Java 代码都可以运行

``` kotlin
import java.util.*

fun demo(source: List<Int>) {
  val list = ArrayList<Int>()
  // 'for'-loops work for Java collections:
  for (item in source)
    list.add(item)
  // Operator conventions work as well:
  for (i in 0..source.size() - 1)
    list[i] = source[i] // get and set are called
}
```

### 返回void的方法

如果一个Java方法返回void，那么在Kotlin中，它会返回`Unit`。
万一有人使用它的返回值，Kotlin的编译器会在调用的地方赋值，因为这个值本身已经提前可以预知了(这个值就是`Unit`)。

### 将Java代码中与Kotlin关键字冲突的标识符进行转义

一些Kotlin的关键字在Java中是合法的标识符: *in*{: .keyword }, *object*{: .keyword }, *is*{: .keyword }, 等等.
如果一个Java库在方法中使用了Kotlin关键字,你仍然可以使用这个方法
使用反引号(`)转义来避免冲突。

``` kotlin
foo.`is`(bar)
```

### Null安全性和平台类型

Java中的所有引用都可能是*null*{: .keyword }值，这使得Kotlin严格的null控制对来自Java的对象来说变得不切实际。在Kotlin中Java声明类型被特别对待叫做*platform types*.这种类型的Null检查是不严格的，所以他们还维持着同Java中一样的安全性。

考虑如下例子:

``` kotlin
val list = ArrayList<String>() // non-null (constructor result)
list.add("Item")
val size = list.size() // non-null (primitive int)
val item = list.get(0) // platform type inferred (ordinary Java object)
```

当我们调用平台类型的变量上的方法时，kotlin不会在编译阶段报出可能为空的错误，但在运行时，会产生空指针异常，或者是断言失败的错误。后者是因为kotlin为了阻止null值传播会生成非空断言语句。

``` kotlin
item.substring(1) // 允许, 如果item为空会抛异常
```

平台类型是*不可转义*的，也就是说我们不能在程序里把他们写出来。
当把一个平台数值赋值给kotlin变量的时候（变量会有一个推断出来的平台类型，上面的例子里就是`item`的类型），我们可以用类型推断，或者指定我们期望的类型（nullable和non-null类型都可以）：

``` kotlin
val nullable: String? = item // 允许，没有问题
val notNull: String = item // 允许，运行时可能失败
```

如果我们指定了一个非空类型，编译器会在赋值前额外生成一个断言。这样kotlin的非空变量就不会有空值。当把平台数值传递给只接受非空数值的kolin函数的时候，也同样会生成这个断言，编译器尽可能的阻止空置在程序里传播。（因为泛型的存在，有时也不能百分百的阻止）

### 平台类型的概念

如上所述，平台类型不能再程序里显式的出现，所以没有针对他们的语法。然而，编译器和IDE有时需要显式他们(如在错误信息，参数信息中)，所以我们用一个好记的标记来表示他们：

* `T!` 表示 "`T` 或者 `T?`"
* `(Mutable)Collection<T>!` 表示 "`T`的java集合，可变的或不可变的，可空的或非空的"
* `Array<(out) T>!` 表示 "`T`(或`T`的子类)的java数组，可空的或非空的"

### 映射类型

Kotlin特殊处理一部分java类型。这些类型不是通过as或is来直接转换，而是_映射_到了指定的kotlin类型上。
映射只发生在编译期间，运行时仍然是原来的类型。（？？）
java的原生类型映射成如下kotlin类型（记得 [平台类型](#platform-types)）：

| **Java类型** | **Kotlin类型**  |
|---------------|------------------|
| `byte`        | `kotlin.Byte`    |
| `short`       | `kotlin.Short`   |
| `int`         | `kotlin.Int`     |
| `long`        | `kotlin.Long`    |
| `char`        | `kotlin.Char`    |
| `float`       | `kotlin.Float`   |
| `double`      | `kotlin.Double`  |
| `boolean`     | `kotlin.Boolean` |
{:.zebra}

一些非原生类型也会作映射：

| **Java类型** | **Kotlin类型**  |
|---------------|------------------|
| `java.lang.Object`       | `kotlin.Any!`    |
| `java.lang.Cloneable`    | `kotlin.Cloneable!`    |
| `java.lang.Comparable`   | `kotlin.Comparable!`    |
| `java.lang.Enum`         | `kotlin.Enum!`    |
| `java.lang.Annotation`   | `kotlin.Annotation!`    |
| `java.lang.Deprecated`   | `kotlin.deprecated!`    |
| `java.lang.Void`         | `kotlin.Nothing!`    |
| `java.lang.CharSequence` | `kotlin.CharSequence!`   |
| `java.lang.String`       | `kotlin.String!`   |
| `java.lang.Number`       | `kotlin.Number!`     |
| `java.lang.Throwable`    | `kotlin.Throwable!`    |
{:.zebra}

集合类型在kotlin里可以是只读的或可变的，因此java集合类型作如下映射：
（下表所有的kotlin类型都在`kotlin`包里）

| **Java类型** | **Kotlin只读类型**  | **Kotlin可变类型** | **加载的平台类型** |
|---------------|------------------|----|----|
| `Iterator<T>`        | `Iterator<T>`        | `MutableIterator<T>`            | `(Mutable)Iterator<T>!`            |
| `Iterable<T>`        | `Iterable<T>`        | `MutableIterable<T>`            | `(Mutable)Iterable<T>!`            |
| `Collection<T>`      | `Collection<T>`      | `MutableCollection<T>`          | `(Mutable)Collection<T>!`          |
| `Set<T>`             | `Set<T>`             | `MutableSet<T>`                 | `(Mutable)Set<T>!`                 |
| `List<T>`            | `List<T>`            | `MutableList<T>`                | `(Mutable)List<T>!`                |
| `ListIterator<T>`    | `ListIterator<T>`    | `MutableListIterator<T>`        | `(Mutable)ListIterator<T>!`        |
| `Map<K, V>`          | `Map<K, V>`          | `MutableMap<K, V>`              | `(Mutable)Map<K, V>!`              |
| `Map.Entry<K, V>`    | `Map.Entry<K, V>`    | `MutableMap.MutableEntry<K,V>` | `(Mutable)Map.(Mutable)Entry<K, V>!` |
{:.zebra}

Java数组的映射在这里提到过 [below](java-interop.html#java-arrays)：

| **Java类型** | **Kotlin类型**  |
|---------------|------------------|
| `int[]`       | `kotlin.IntArray!` |
| `String[]`    | `kotlin.Array<(out) String>!` |
{:.zebra}


### Kotlin中的Java泛型

Kotlin的泛型和Java的有些不同（详见 [Generics](generics.html)）。当引入java类型的时候，我们作如下转换：

* Java的通配符转换成类型投射
  * `Foo<? extends Bar>` 转换成 `Foo<out Bar!>!`
  * `Foo<? super Bar>` 转换成 `Foo<in Bar!>!` 

* Java的原始类型转换成星号投射
  * `List` 转换成 `List<*>!`, 也就是 `List<out Any?>!`

和Java一样，Kotlin在运行时不保留泛型，即对象不知道传递到他们构造器中的那些参数的的实际类型。
也就是说，运行时无法区分`ArrayList<Integer>()` 和 `ArrayList<Character>()`.
这意味着，不可能用 *is*{: .keyword }-来检测泛型。
Kotlin只允许用*is*{: .keyword }-来检测星号投射的泛型类型:

``` kotlin
if (a is List<Int>) // 错误: 不能检测是否是一个Int的List
// but
if (a is List<*>) // 可以：不保证list里面的内容类型
```

（？？？）
Like Java's, Kotlin's generics are not retained at runtime, i.e. objects do not carry information about actual type arguments passed to their constructors,
i.e. `ArrayList<Integer>()` is indistinguishable from `ArrayList<Character>()`.
This makes it impossible to perform *is*{: .keyword }-checks that take generics into account.
Kotlin only allows *is*{: .keyword }-checks for star-projected generic types:

``` kotlin
if (a is List<Int>) // Error: cannot check if it is really a List of Ints
// but
if (a is List<*>) // OK: no guarantees about the contents of the list
```

### Java数组

和Java不同，Kotlin里的数组不是协变的。Kotlin不允许我们把`Array<String>` 赋值给 `Array<Any>`，从而避免了可能的运行时错误。Kotlin也禁止我们把一个子类的数组当做父类的数组传递进Kotlin的方法里。
但是对Java方法，这是允许的（考虑这种形式的平台类型[platform types](#platform-types) `Array<(out) String>!`）。

Java平台上，原生数据类型的数组被用来避免封箱/开箱的操作开销。
由于Kotlin隐藏了这些实现细节，就得有一个变通方法和Java代码交互。
每个原生类型的数组都有一个特有类(specialized class)来处理这种问题(`IntArray`, `DoubleArray`, `CharArray` ...)。
它们不是`Array`类，而是被编译成java的原生数组，来获得最好的性能。

假设有一个Java方法，它接收一个表示索引的int数组作参数

``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // code here...
    }
}
```

在Kotlin里你可以这样传递一个原生数组:

``` kotlin
val javaObj = JavaArrayExample()
val array = intArray(0, 1, 2, 3)
javaObj.removeIndices(array)  // passes int[] to method
```

Java类也会这样声明方法，表示参数是可变参数。

``` java
public class JavaArrayExample {

    public void removeIndices(int... indices) {
        // code here...
    }
}
```

这种情况，你需要用展开操作符 `*` 来传递 `IntArray`：

``` kotlin
val javaObj = JavaArray()
val array = intArray(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

目前无法传递 *null*{: .keyword } 给一个变参的方法。

当编译成jvm字节码的时候，编译器会优化对数组的访问，确保不会产生额外的负担。

``` kotlin
val array = array(1, 2, 3, 4)
array[x] = array[x] * 2 // 不会生成对get() 和 set()的调用
for (x in array) // 不会创建迭代器
  print(x)
```

即便是用索引遍历数组。

``` kotlin
for (i in array.indices) // 不会创建迭代器
  array[i] += 2
```

最后，*in*{: .keyword }-检测也没有额外负担。

``` kotlin
if (i in array.indices) { // 和 (i >= 0 && i < array.size) 一样
  print(array[i])
}
```

### 受检异常

在Kotlin里，所有的异常都是非受检的, 也就是说，编译器不会强制你去捕捉任何异常。
因此，你调用一个声明了异常的java方法的时候，kotlin不会强制你作处理。

``` kotlin
fun render(list: List<*>, to: Appendable) {
  for (item in list)
    to.append(item.toString()) // Java里会让你在这里捕捉IOException
}
```

### 对象方法

当java类型被引入到kotlin里时，所有的`java.lang.Object`类型引用，会被转换成 `Any`。
因为`Any`不是平台独有的，它仅声明了三个成员方法：`toString()`, `hashCode()` 和 `equals()`，所以为了能用到`java.lang.Object`的其他方法，kotlin采用了[扩展函数](extensions.html)。

#### wait()/notify()

[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 第69条善意的提醒了要用concurrency类而不是`wait()` 和 `notify()`。
因此，`Any`不提供这两个方法。
你一定要用的话，就把它转换成`java.lang.Object`。

```kotlin
(foo as java.lang.Object).wait()
```

#### getClass()

获取一个对象的类型信息，我们可以用javaClass这个扩展属性。

``` kotlin
val fooClass = foo.javaClass
```

用javaClass<Foo>()，而不是java里的写法`Foo.class`。

``` kotlin
val fooClass = javaClass<Foo>()
```

#### clone()

要重写`clone()`，扩展`kotlin.Cloneable`：

```kotlin

class Example : Cloneable {
  override fun clone(): Any { ... }
}
```

不要忘了 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html),第11条: *谨慎的重写克隆*。

#### finalize()

要重载 `finalize()`, 你要做的仅仅是声明它，不需要 *override*{:.keyword} 关键字：

```kotlin
class C {
  protected fun finalize() {
    // 具体逻辑
  }
}
```

根据java的规则， `finalize()`不能为 *private*{: .keyword }。

### java类的继承
在kotlin里，超类里最多只能有一个java类(java接口数目不限)。这个java类必须放在超类列表的最前面。

### 访问静态成员

java类的静态成员就是它们的 “同伴对象”。我们无法将这样的“同伴对象”当作数值来传递，
但可以显式的访问它们，比如：

``` kotlin
if (Character.isLetter(a)) {
  // ...
}
```

### Java 反射

Java反射可以用在kotlin类上，反之亦然。前面提过，你可以 `instance.javaClass` 或者 `javaClass<ClassName>()` 开始基于 `java.lang.Class` 的java反射操作。你也可以通过调用 `.kotlin` 使用kotlin的反射。

``` kotlin 
val kClass = x.javaClass.kotlin  
```

类似的，你可以把kotlin反射转换成java反射： `ClassName::class.java` 和 `javaClass<ClassName>()`都可以。
其他支持的情况包括 获取一个java的getter/setter方法，或者一个kotlin属性的 backing field，getting a containing `KPackage` instance for a Java class, and getting a `KProperty` for a Java field. (???)

### SAM(单抽象方法) 转换

就像java8那样，kotlin支持SAM转换，这意味着kotlin函数字面量可以被自动的转换成只有一个非默认方法的java接口的实现，只要这个方法的参数类型能够跟这个kotlin函数的参数类型匹配的上。

你可以这样创建SAM接口的实例：

``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

...在方法调用里:

``` kotlin
val executor = ThreadPoolExecutor()
// Java签名: void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```

如果java类有多个接受函数接口的方法，你可以用一个适配函数来把闭包转成你需要的SAM类型。编译器也会在必要时生成这些适配函数。

``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```

注意SAM的转换只对接口有效，对抽象类无效，即使它们就只有一个抽象方法。

还要注意这个特性只针对和java的互操作；因为kotlin有合适的函数类型，把函数自动转换成kotlin接口的实现是没有必要的，也就没有支持了。

## Java调用Kotlin代码

Java可以轻松调用Kotlin代码。

### 包级别的函数

`org.foo.bar`包里声明的所有的函数和属性，都会被放到一个叫`org.foo.bar.BarPackage`的java类里。

``` kotlin
package demo

class Foo

fun bar() {
}

```

``` java
// Java
new demo.Foo();
demo.DemoPackage.bar();
```

对于最外层的包（java里叫做缺省包），创建一个叫做`_DefaultPackage`的类。

### 静态方法和字段

上面说过，kotlin把包级别的函数生成为静态方法。此外，还会把类的命名对象或伙伴对象中有`@platformStatic`标记的函数也生成为静态方法。比如：

``` kotlin
class C {
  companion object {
    platformStatic fun foo() {}
    fun bar() {}
  }
}
```

现在，`foo()`在java里就是静态的了，而`bar()` 不是：

``` java
C.foo(); // 没问题
C.bar(); // 错误: 不是一个静态方法
```

同样的，命名对象：

``` kotlin
object Obj {
    platformStatic fun foo() {}
    fun bar() {}
}
```

java里：

``` java
Obj.foo(); // 没问题
Obj.bar(); // 错误
Obj.INSTANCE$.bar(); // 对单例的方法调用
Obj.INSTANCE$.foo(); // 也行
```

命名对象和伙伴对象里的公开属性，还有顶层的有 `const` 标记的属性，
会被转成java中的静态字段：

``` kotlin
// file example.kt

object Obj {
  val CONST = 1
}

const val MAX = 239
```

java里：

``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
```

### 用@platformName解决签名冲突

有时我们想让一个kotlin里的命名函数在字节码里有另外一个jvm名字。
最突出的例子来自于 *类型擦除*:

``` kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```

这两个函数不能同时定义，因为它们的jvm签名是一样的：
`filterValid(Ljava/util/List;)Ljava/util/List;`.
如果我们真的相让它们在kotlin里用同一个名字，我们需要用`@platformName`去注释它们中的一个（或两个），指定的另外一个名字当参数：

``` kotlin
fun List<String>.filterValid(): List<String>
@platformName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```

在kotlin里它们可以都用`filterValid`来访问，但是在java里，它们是`filterValid` 和 `filterValidInt`.

同样的技巧也适用于属性 `x` 和函数 `getX()` 共存：

``` kotlin
val x: Int
  @platformName("getX_prop")
  get() = 15

fun getX() = 10
```

### 重载生成

通常，如果你写一个有默认参数值的kotlin方法，在java里，只会有一个有完整参数的签名。如果你要暴露多个重载给java调用者，你可以使用@jvmOverloads标记。

``` kotlin
jvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
    ...
}
```

对于每一个有默认值的参数，都会生成一个额外的重载，这个重载会把这个参数和它右边的所有参数都移除掉。在上面这个例子里，生成下面的方法：

``` java
// Java
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```

构造函数，静态函数等也能用这个标记。但他不能用在抽象方法上，包括接口中的方法。

注意一下，[Secondary Constructors](classes.html#secondary-constructors) 描述过，如果一个类的所有构造函数参数都有默认值，会生成一个公开的无参构造函数。这就算没有@jvmOverloads标记也有效。

### 受检异常

上面说过，kotlin没有受检异常。
所以，通常，kotlin函数的java签名没有声明抛出异常。
于是如果我们有一个kotlin函数：

``` kotlin
package demo

fun foo() {
  throw IOException()
}
```

然后我们想要在java里调用它，捕捉这个异常：

``` java
// Java
try {
  demo.DemoPackage.foo();
}
catch (IOException e) { // 错误: foo() 没有声明 IOException
  // ...
}
```

因为`foo()`没有声明 `IOException`，java编译器报了错误信息。
为了解决这个问题，要在kotlin里使用`@throws`标记。

``` kotlin
@throws(IOException::class) fun foo() {
    throw IOException()
}
```

### Null安全性

当从java中调用kotlin函数时，没人阻止我们传递 *null*{: .keyword } 给一个非空参数。
这就是为什么kotlin给所有期望非空参数的公开函数生成运行时检测。
这样我们就能在java代码里立即得到 `NullPointerException`。

### 属性

属性getters被转换成 *get*-方法，setters转换成*set*-方法。
