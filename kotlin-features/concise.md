### 简洁

使用一行代码创建一个包含getters, setters, `equals()`, `hashCode()`, `toString()` and `copy()` 的POJO

``` kotlin
data class Customer(val name: String, val email: String, val company: String)
```

或者使用lambda表达式来过滤链表：

``` kotlin
val positiveNumbers = list.filter {it > 0}
```

想要单例？创建object就可以了：

``` kotlin
object ThisIsASingleton {
    val companyName: String = "JetBrains"
}
```
