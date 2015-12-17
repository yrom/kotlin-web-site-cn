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

当我们调用平台类型的变量上的方法时，编译阶段kotlin不会报出可能为空的错误，但在运行时，会产生空指针异常，或者是断言失败的错误。后者是因为kotlin为了阻止null值继续被使用会生成非空断言语句。
When we call methods on variables of platform types, Kotlin does not issue nullability errors at compile time,
but the call may fail at runtime, because of a null-pointer exception or an assertion that Kotlin generates to
prevent nulls from propagating:

``` kotlin
item.substring(1) // allowed, may throw an exception if item == null
```

平台类型是*不可转义*的，也就是说我们不能在程序里把他们写出来。
当把一个平台数值赋值给kotlin变量的时候（变量会有一个推断出来的平台类型，上面的例子里就是`item`的类型），我们可以用类型推断，或者指定我们期望的类型（nullable和non-null类型都可以）
Platform types are *non-denotable*, meaning that one can not write them down explicitly in the language.
When a platform value is assigned to a Kotlin variable, we can rely on type inference (the variable will have an inferred platform type then,
 as `item` has in the example above), or we can choose the type that we expect (both nullable and non-null types are allowed):

``` kotlin
val nullable: String? = item // allowed, always works
val notNull: String = item // allowed, may fail at runtime
```

如果我们指定了一个非空类型，编译器会在赋值前额外生成一个断言。这样kotlin的非空变量就不会有空值。当把平台数值传递给只接受非空数值的kolin函数的时候，也同样会生成这个断言，编译器尽可能的阻止空置在程序里传播。（因为泛型的存在，有时也不能百分百的阻止）
If we choose a non-null type, the compiler will emit an assertion upon assignment. This prevents Kotlin's non-null variables from holding
nulls. Assertions are also emitted when we pass platform values to Kotlin functions expecting non-null values etc.
Overall, the compiler does its best to prevent nulls from propagating far through the program (although sometimes this is
impossible to eliminate entirely, because of generics).

### 平台类型的概念
#### Notation for Platform Types

如上所述，平台类型不能再程序里显式的出现，所以语言没有针对他们的语法。然而，编译器和IDE有时需要显式他们(如在错误信息，参数信息中)，所以我们用一个好记的标记来表示他们：
As mentioned above, platform types cannot be mentioned explicitly in the program, so there's no syntax for them in the language.
Nevertheless, the compiler and IDE need to display them sometimes (in error messages, parameter info etc), so we have a
mnemonic notation for them:

* `T!` 表示 "`T` 或者 `T?`"
* `(Mutable)Collection<T>!` 表示 "`T`的java集合，可变的或不可变的，可空的或非空的"
* `Array<(out) T>!` 表示 "`T`(或`T`的子类)的java数组，可空的或非空的"
* `T!` means "`T` or `T?`",
* `(Mutable)Collection<T>!` means "Java collection of `T` may be mutable or not, may be nullable or not",
* `Array<(out) T>!` means "Java array of `T` (or a subtype of `T`), nullable or not"

### 映射类型
### Mapped types

Kotlin特殊处理一部分java类型。这些类型不是通过as或is来直接转换，而是_映射_到了指定的kotlin类型上。
映射只发生在编译期间，运行时仍然是原来的类型。（？？）
java的原生类型映射成如下kotlin类型（记得 [platform types](#platform-types)）：
Kotlin treats some Java types specially. Such types are not loaded from Java "as is", but are _mapped_ to corresponding Kotlin types.
The mapping only matters at compile time, the runtime representation remains unchanged.
 Java's primitive types are mapped to corresponding Kotlin types (keeping [platform types](#platform-types) in mind):

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

| **Java type** | **Kotlin type**  |
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
Some non-primitive built-in classes are also mapped:


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

| **Java type** | **Kotlin type**  |
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
Collection types may be read-only or mutable in Kotlin, so Java's collections are mapped as follows
(all Kotlin types in this table reside in the package `kotlin`):

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

| **Java type** | **Kotlin read-only type**  | **Kotlin mutable type** | **Loaded platform type** |
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
Java's arrays are mapped as mentioned [below](java-interop.html#java-arrays):

| **Java类型** | **Kotlin类型**  |
|---------------|------------------|
| `int[]`       | `kotlin.IntArray!` |
| `String[]`    | `kotlin.Array<(out) String>!` |
{:.zebra}

| **Java type** | **Kotlin type**  |
|---------------|------------------|
| `int[]`       | `kotlin.IntArray!` |
| `String[]`    | `kotlin.Array<(out) String>!` |
{:.zebra}

### Kotlin中的Java泛型
### Java generics in Kotlin

Kotlin的泛型和Java的有些不同（详见 [Generics](generics.html)）。当引入java类型的时候，我们作如下转换：
Kotlin's generics are a little different from Java's (see [Generics](generics.html)). When importing Java types to Kotlin we perform some conversions:

* Java的通配符转换成类型投射
  * `Foo<? extends Bar>` 转换成 `Foo<out Bar!>!`
  * `Foo<? super Bar>` 转换成 `Foo<in Bar!>!` 
* Java's wildcards are converted into type projections
  * `Foo<? extends Bar>` becomes `Foo<out Bar!>!`
  * `Foo<? super Bar>` becomes `Foo<in Bar!>!`

* Java的原始类型转换成星号投射
  * `List` 转换成 `List<*>!`, 也就是 `List<out Any?>!`
* Java's raw types are converted into star projections
  * `List` becomes `List<*>!`, i.e. `List<out Any?>!`

和Java一样，Kotlin在运行时不保留泛型，即对象不知道传递到他们构造器中的那些参数的的实际类型。
也就是说，运行时无法区分`ArrayList<Integer>()` 和 `ArrayList<Character>()`.
这意味着，不可能用 *is*{: .keyword }-来检测泛型。
Kotlin只允许用*is*{: .keyword }-来检测星号投射的泛型类型:

``` kotlin
if (a is List<Int>) // 错误: 不能检测是否是一个Int的List
// but
if (a is List<*>) // 可以：不保证list里面的内容类型
```

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
### Java Arrays

和Java不同，Kotlin里的数组不是协变的。Kotlin不允许我们把`Array<String>` 赋值给 `Array<Any>`，从而避免了可能的运行时错误。Kotlin也禁止我们把一个子类的数组当做父类的数组传递进Kotlin的方法里。
但是对Java方法，这是允许的（考虑这种形式的平台类型[platform types](#platform-types) `Array<(out) String>!`）。
Arrays in Kotlin are invariant, unlike Java. This means that Kotlin does not let us assign an `Array<String>` to an `Array<Any>`,
which prevents a possible runtime failure. Passing an array of a subclass as an array of superclass to a Kotlin method is also prohibited,
but for Java methods this is allowed (though [platform types](#platform-types) of the form `Array<(out) String>!`).

Java平台上，原生数据类型的数组被用来避免封箱/开箱的操作开销。
由于Kotlin隐藏了这些实现细节，就得有一个变通方法和Java代码交互。（？？？）
每个原生类型的数组都有一个特有类(specialized class)来处理这种问题(`IntArray`, `DoubleArray`, `CharArray` ...)。
它们不是`Array`类，而是被编译成java的原生数组，来获得最好的性能。
Arrays are used with primitive datatypes on the Java platform to avoid the cost of boxing/unboxing operations.
As Kotlin hides those implementation details, a workaround is required to interface with Java code.
There are specialized classes for every type of primitive array (`IntArray`, `DoubleArray`, `CharArray`, and so on) to handle this case.
They are not related to the `Array` class and are compiled down to Java's primitive arrays for maximum performance.

假设有一个Java方法，它接收一个表示索引的int数组作参数
Suppose there is a Java method that accepts an int array of indices:

``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // code here...
    }
}
```
``` java
public class JavaArrayExample {

    public void removeIndices(int[] indices) {
        // code here...
    }
}
```

在Kotlin里你可以这样传递一个原生数组:
To pass an array of primitive values you can do the following in Kotlin:

``` kotlin
val javaObj = JavaArrayExample()
val array = intArray(0, 1, 2, 3)
javaObj.removeIndices(array)  // passes int[] to method
```
``` kotlin
val javaObj = JavaArrayExample()
val array = intArray(0, 1, 2, 3)
javaObj.removeIndices(array)  // passes int[] to method
```

Java类也会这样声明方法，表示索引的参数是可变参数。
Java classes sometimes use a method declaration for the indices with a variable number of arguments (varargs).

``` java
public class JavaArrayExample {

    public void removeIndices(int... indices) {
        // code here...
    }
}
```
``` java
public class JavaArrayExample {

    public void removeIndices(int... indices) {
        // code here...
    }
}
```

这种情况，你需要用展开操作符 `*` 来传递 `IntArray`：
In that case you need to use the spread operator `*` to pass the `IntArray`:

``` kotlin
val javaObj = JavaArray()
val array = intArray(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```
``` kotlin
val javaObj = JavaArray()
val array = intArray(0, 1, 2, 3)
javaObj.removeIndicesVarArg(*array)
```

目前无法传递 *null*{: .keyword } 给一个变参的方法。
It's currently not possible to pass *null*{: .keyword } to a method that is declared as varargs.

当编译成jvm字节码的时候，编译器会优化对数组的访问，确保不会产生额外的负担。
When compiling to JVM byte codes, the compiler optimizes access to arrays so that there's no overhead introduced:

``` kotlin
val array = array(1, 2, 3, 4)
array[x] = array[x] * 2 // no actual calls to get() and set() generated
for (x in array) // no iterator created
  print(x)
```
``` kotlin
val array = array(1, 2, 3, 4)
array[x] = array[x] * 2 // no actual calls to get() and set() generated
for (x in array) // no iterator created
  print(x)
```

即便是用索引遍历数组。
Even when we navigate with an index, it does not introduce any overhead

``` kotlin
for (i in array.indices) // 不会创建迭代器
  array[i] += 2
```
``` kotlin
for (i in array.indices) // no iterator created
  array[i] += 2
```

最后，*in*{: .keyword }-检测也没有额外负担。
Finally, *in*{: .keyword }-checks have no overhead either

``` kotlin
if (i in array.indices) { // 和 (i >= 0 && i < array.size) 一样
  print(array[i])
}
```
``` kotlin
if (i in array.indices) { // same as (i >= 0 && i < array.size)
  print(array[i])
}
```

### 受检异常
### Checked Exceptions

在Kotlin里，所有的异常都是非受检的, 也就是说，编译器不会强制你去捕捉任何异常。
因此，你调用一个声明了异常的java方法的时候，kotlin不会强制你作处理。
In Kotlin, all exceptions are unchecked, meaning that the compiler does not force you to catch any of them.
So, when you call a Java method that declares a checked exception, Kotlin does not force you to do anything:

``` kotlin
fun render(list: List<*>, to: Appendable) {
  for (item in list)
    to.append(item.toString()) // Java里会让你在这里捕捉IOException
}
```
``` kotlin
fun render(list: List<*>, to: Appendable) {
  for (item in list)
    to.append(item.toString()) // Java would require us to catch IOException here
}
```

### 对象方法
### Object Methods

当java类型被引入到kotlin里时，所有的`java.lang.Object`类型引用，会被转换成 `Any`。
因为`Any`不是平台独有的，它仅声明了三个成员方法：`toString()`, `hashCode()` 和 `equals()`，所以为了能用到`java.lang.Object`的其他方法，kotlin采用了[extension functions](extensions.html)。
When Java types are imported into Kotlin, all the references of the type `java.lang.Object` are turned into `Any`.
Since `Any` is not platform-specific, it only declares `toString()`, `hashCode()` and `equals()` as its members,
so to make other members of `java.lang.Object` available, Kotlin uses [extension functions](extensions.html).

#### wait()/notify()
#### wait()/notify()

[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) 第69条善意的提醒了要用concurrency类而不是`wait()` 和 `notify()`。
因此，`Any`不提供这两个方法。
你一定要用的话，就把它转换成`java.lang.Object`。
[Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html) Item 69 kindly suggests to prefer concurrency utilities to `wait()` and `notify()`.
Thus, these methods are not available on references of type `Any`.
If you really need to call them, you can cast to `java.lang.Object`:

```kotlin
(foo as java.lang.Object).wait()
```
```kotlin
(foo as java.lang.Object).wait()
```

#### getClass()
#### getClass()

获取一个对象的类型信息，我们可以用javaClass这个扩展属性。
To retrieve the type information from an object, we use the javaClass extension property.

``` kotlin
val fooClass = foo.javaClass
```
``` kotlin
val fooClass = foo.javaClass
```

用javaClass<Foo>()，而不是java里的写法`Foo.class`。
Instead of Java's `Foo.class` use javaClass<Foo>().

``` kotlin
val fooClass = javaClass<Foo>()
```
``` kotlin
val fooClass = javaClass<Foo>()
```

#### clone()
#### clone()

要重载`clone()`，扩展`kotlin.Cloneable`：
To override `clone()`, your class needs to extend `kotlin.Cloneable`:

```kotlin

class Example : Cloneable {
  override fun clone(): Any { ... }
}
```
```kotlin

class Example : Cloneable {
  override fun clone(): Any { ... }
}
```

不要忘了 [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html),第11条: *谨慎的重载克隆*。
 Do not forget about [Effective Java](http://www.oracle.com/technetwork/java/effectivejava-136174.html), Item 11: *Override clone judiciously*.

#### finalize()
#### finalize()

要重载 `finalize()`, 你要做的仅仅是声明它，不需要 *override*{:.keyword} 关键字：
To override `finalize()`, all you need to do is simply declare it, without using the *override*{:.keyword} keyword:

```kotlin
class C {
  protected fun finalize() {
    // 具体逻辑
  }
}
```
```kotlin
class C {
  protected fun finalize() {
    // finalization logic
  }
}
```

根据java的规则， `finalize()`不能为 *private*{: .keyword }。
According to Java's rules, `finalize()` must not be *private*{: .keyword }.

### java类的继承
### Inheritance from Java classes
在kotlin里，超类里最多只能有一个java类(java接口数目不限)。这个java类必须放在超类列表的最前面。
At most one Java-class (and as many Java interfaces as you like) can be a supertype for a class in Kotlin. This class must go first in the supertype list.

### 访问静态成员
### Accessing static members

java类的静态成员就是它们的 “同伴对象”。我们无法将这样的“同伴对象”当作数值来传递，
但可以显式的访问它们，比如：
Static members of Java classes form "companion objects" for these classes. We cannot pass such a "companion object" around as a value,
but can access the members explicitly, for example

``` kotlin
if (Character.isLetter(a)) {
  // ...
}
```
``` kotlin
if (Character.isLetter(a)) {
  // ...
}
```

### Java 反射
### Java Reflection

Java反射可以用在kotlin类上，反之亦然。前面提过，你可以 `instance.javaClass` 或者 `javaClass<ClassName>()` 开始基于 `java.lang.Class` 的java反射操作。你也可以通过调用 `.kotlin` 使用kotlin的反射。
Java reflection works on Kotlin classes and vice versa. As mentioned above, you can use `instance.javaClass` or 
`javaClass<ClassName>()` to enter Java reflection through `java.lang.Class`. You can then "convert" to Kotlin reflection
by calling `.kotlin`:
 
``` kotlin 
val kClass = x.javaClass.kotlin  
```
``` kotlin 
val kClass = x.javaClass.kotlin  
```
 
类似的，你可以把kotlin反射转换成java反射： `ClassName::class.java` 和 `javaClass<ClassName>()`都可以。
其他支持的情况包括 获取一个java的getter/setter方法，或者一个kotlin属性的 backing field，getting a containing `KPackage` instance for a Java class, and getting a `KProperty` for a Java field. (???)
In much the same way you can convert from Kotlin reflection to Java: `ClassName::class.java` is the same as `javaClass<ClassName>()`.
Other supported cases include acquiring a Java getter/setter method or a backing field for a Kotlin property, 
getting a containing `KPackage` instance for a Java class, and getting a `KProperty` for a Java field.

### SAM(单抽象方法) 转换
### SAM Conversions

就像java8那样，kotlin支持SAM转换，这意味着kotlin函数字面量可以被自动的转换成只有一个非默认方法的java接口的实现，只要这个方法的参数类型能够跟这个kotlin函数的参数类型匹配的上。
Just like Java 8, Kotlin supports SAM conversions. This means that Kotlin function literals can be automatically converted
into implementations of Java interfaces with a single non-default method, as long as the parameter types of the interface
method match the parameter types of the Kotlin function.

你可以这样创建SAM接口的实例：
You can use this for creating instances of SAM interfaces:

``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```
``` kotlin
val runnable = Runnable { println("This runs in a runnable") }
```

...在方法调用里:
...and in method calls:

``` kotlin
val executor = ThreadPoolExecutor()
// Java签名: void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```
``` kotlin
val executor = ThreadPoolExecutor()
// Java signature: void execute(Runnable command)
executor.execute { println("This runs in a thread pool") }
```

如果java类有多个接受函数接口的方法，你可以用一个适配函数来把闭包转成你需要的SAM类型。编译器也会在必要时生成这些适配函数。
If the Java class has multiple methods taking functional interfaces, you can choose the one you need to call by
using an adapter function that converts a lambda to a specific SAM type. Those adapter functions are also generated
by the compiler when needed.

``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```
``` kotlin
executor.execute(Runnable { println("This runs in a thread pool") })
```

注意SAM的转换只对接口有效，对抽象类无效，即使它们就只有一个抽象方法。
Note that SAM conversions only work for interfaces, not for abstract classes, even if those also have just a single
abstract method.

还要注意这个特性只针对和java的互操作；因为kotlin有合适的函数类型，把函数自动转换成kotlin接口的实现是没有必要的，也就没有支持了。
Also note that this feature works only for Java interop; since Kotlin has proper function types, automatic conversion
of functions into implementations of Kotlin interfaces is unnecessary and therefore unsupported.

## Java调用Kotlin代码
## Calling Kotlin code from Java

Java可以轻松调用Kotlin代码。
Kotlin code can be called from Java easily.

### 包级别的函数
### Package-Level Functions

`org.foo.bar`包里声明的所有的函数和属性，都会被放到一个叫`org.foo.bar.BarPackage`的java类里。
All the functions and properties declared inside a package `org.foo.bar` are put into a Java class named `org.foo.bar.BarPackage`.

``` kotlin
package demo

class Foo

fun bar() {
}

```
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
``` java
// Java
new demo.Foo();
demo.DemoPackage.bar();
```

对于最外层的包（java里叫做缺省包），创建一个叫做`_DefaultPackage`的类。
For the root package (the one that's called a "default package" in Java), a class named `_DefaultPackage` is created.

### 静态方法和字段
### Static Methods and Fields

上面说过，kotlin把包级别的函数生成为静态方法。此外，还会把类的命名对象或伙伴对象中有`@platformStatic`标记的函数也生成为静态方法。比如：
As mentioned above, Kotlin generates static methods for package-level functions. On top of that, it also generates static methods
for functions defined in named objects or companion objects of classes and annotated as `@platformStatic`. For example:

``` kotlin
class C {
  companion object {
    platformStatic fun foo() {}
    fun bar() {}
  }
}
```
``` kotlin
class C {
  companion object {
    platformStatic fun foo() {}
    fun bar() {}
  }
}
```

现在，`foo()`在java里就是静态的了，而`bar()` 不是：
Now, `foo()` is static in Java, while `bar()` is not:

``` java
C.foo(); // 没问题
C.bar(); // 错误: 不是一个静态方法
```
``` java
C.foo(); // works fine
C.bar(); // error: not a static method
```

同样的，命名对象：
Same for named objects:

``` kotlin
object Obj {
    platformStatic fun foo() {}
    fun bar() {}
}
```
``` kotlin
object Obj {
    platformStatic fun foo() {}
    fun bar() {}
}
```

java里：
In Java:

``` java
Obj.foo(); // 没问题
Obj.bar(); // 错误
Obj.INSTANCE$.bar(); // 对单例的方法调用
Obj.INSTANCE$.foo(); // 也行
```
``` java
Obj.foo(); // works fine
Obj.bar(); // error
Obj.INSTANCE$.bar(); // works, a call through the singleton instance
Obj.INSTANCE$.foo(); // works too
```

命名对象和伙伴对象里的公开属性，还有顶层的有 `const` 标记的属性，
会被转成java中的静态字段：
Also, public properties defined in objects and companion objects, as well as top-level properties annotated with `const`,
are turned into static fields in Java:

``` kotlin
// file example.kt

object Obj {
  val CONST = 1
}

const val MAX = 239
```
``` kotlin
// file example.kt

object Obj {
  val CONST = 1
}

const val MAX = 239
```

java里：
In Java:

``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
```
``` java
int c = Obj.CONST;
int d = ExampleKt.MAX;
```

### 用@platformName解决签名冲突
### Handling signature clashes with @platformName

有时我们想让一个kotlin里的命名函数在字节码里有另外一个jvm名字。
最突出的例子来自于 *类型擦除*:
Sometimes we have a named function in Kotlin, for which we need a different JVM name the byte code.
The most prominent example happens due to *type erasure*:

``` kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```
``` kotlin
fun List<String>.filterValid(): List<String>
fun List<Int>.filterValid(): List<Int>
```

这两个函数不能同时定义，因为它们的jvm签名是一样的：
`filterValid(Ljava/util/List;)Ljava/util/List;`.
如果我们真的相让它们在kotlin里用同一个名字，我们需要用`@platformName`去注释它们中的一个（或两个），指定的另外一个名字当参数：
These two functions can not be defined side-by-side, because their JVM signatures are the same: `filterValid(Ljava/util/List;)Ljava/util/List;`.
If we really want them to have the same name in Kotlin, we can annotate one (or both) of them with `@platformName` and specify a different name as an argument:

``` kotlin
fun List<String>.filterValid(): List<String>
@platformName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```
``` kotlin
fun List<String>.filterValid(): List<String>
@platformName("filterValidInt")
fun List<Int>.filterValid(): List<Int>
```

在kotlin里它们可以都用`filterValid`来访问，但是在java里，它们是`filterValid` 和 `filterValidInt`.
From Kotlin they will be accessible by the same name `filterValid`, but from Java it will be `filterValid` and `filterValidInt`.

同样的技巧也适用于属性 `x` 和函数 `getX()` 共存：
The same trick applies when we need to have a property `x` alongside with a function `getX()`:

``` kotlin
val x: Int
  @platformName("getX_prop")
  get() = 15

fun getX() = 10
```
``` kotlin
val x: Int
  @platformName("getX_prop")
  get() = 15

fun getX() = 10
```

### 重载生成
### Overloads Generation

通常，如果你写一个有默认参数值的kotlin方法，在java里，只会有一个有完整参数的签名。如果你要暴露多个重载给java调用者，你可以使用@jvmOverloads标记。
Normally, if you write a Kotlin method with default parameter values, it will be visible in Java only as a full
signature, with all parameters present. If you wish to expose multiple overloads to Java callers, you can use the
@jvmOverloads annotation.


``` kotlin
jvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
    ...
}
```
``` kotlin
jvmOverloads fun f(a: String, b: Int = 0, c: String = "abc") {
    ...
}
```

对于每一个有默认值的参数，都会生成一个额外的重载，这个重载会把这个参数和它右边的所有参数都移除掉。在上面这个例子里，生成下面的方法。
For every parameter with a default value, this will generate one additional overload, which has this parameter and
all parameters to the right of it in the parameter list removed. In this example, the following methods will be
generated:

``` java
// Java
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```
``` java
// Java
void f(String a, int b, String c) { }
void f(String a, int b) { }
void f(String a) { }
```

构造函数，静态函数等也能用这个标记。但他不能用在抽象方法上，包括接口中的方法。
The annotation also works for constructors, static methods etc. It can't be used on abstract methods, including methods
defined in interfaces.

注意一下，[Secondary Constructors](classes.html#secondary-constructors) 描述过，如果一个类的所有构造函数参数都有默认值，会生成一个公开的无参构造函数。这就算没有@jvmOverloads标记也有效。
Note that, as described in [Secondary Constructors](classes.html#secondary-constructors), if a class has default
values for all constructor parameters, a public no-argument constructor will be generated for it. This works even
if the @jvmOverloads annotation is not specified.

### 受检异常
### Checked Exceptions

上面说过，kotlin没有受检异常。
所以，通常，kotlin函数的java签名没有声明抛出异常。
于是如果我们有一个kotlin函数：
As we mentioned above, Kotlin does not have checked exceptions.
So, normally, the Java signatures of Kotlin functions do not declare exceptions thrown.
Thus if we have a function in Kotlin like this:

``` kotlin
package demo

fun foo() {
  throw IOException()
}
```
``` kotlin
package demo

fun foo() {
  throw IOException()
}
```

然后我们想要在java里调用它，捕捉这个异常：
And we want to call it from Java and catch the exception:

``` java
// Java
try {
  demo.DemoPackage.foo();
}
catch (IOException e) { // 错误: foo() 没有声明 IOException
  // ...
}
```
``` java
// Java
try {
  demo.DemoPackage.foo();
}
catch (IOException e) { // error: foo() does not declare IOException in the throws list
  // ...
}
```

因为`foo()`没有声明 `IOException`，java编译器报了错误信息。
为了解决这个问题，要在kotlin里使用`@throws`标记。
we get an error message from the Java compiler, because `foo()` does not declare `IOException`.
To work around this problem, use the `@throws` annotation in Kotlin:

``` kotlin
@throws(IOException::class) fun foo() {
    throw IOException()
}
```
``` kotlin
@throws(IOException::class) fun foo() {
    throw IOException()
}
```

### Null安全
### Null-safety

当从java中调用kotlin函数时，没人阻止我们传递 *null*{: .keyword } 给一个非空参数。
这就是为什么kotlin给所有期望非空参数的公开函数生成运行时检测。
这样我们就能在java代码里立即得到 `NullPointerException`。
When calling Kotlin functions from Java, nobody prevents us from passing *null*{: .keyword } as a non-null parameter.
That's why Kotlin generates runtime checks for all public functions that expect non-nulls.
This way we get a `NullPointerException` in the Java code immediately.

### 属性
### Properties

属性getters被转换成 *get*-方法，setters转换成*set*-方法。
Property getters are turned into *get*-methods, and setters – into *set*-methods.
