### 简洁

使用一行代码创建一个包含 getters、 setters、 `equals()`、 `hashCode()`、 `toString()` 以及 `copy()` 的 POJO：

``` kotlin
data class Customer(val name: String, val email: String, val company: String)
```

或者使用 lambda 表达式来过滤列表：

``` kotlin
val positiveNumbers = list.filter {it > 0}
```

想要单例？创建一个 object 就可以了：

``` kotlin
object ThisIsASingleton {
    val companyName: String = "JetBrains"
}
```
