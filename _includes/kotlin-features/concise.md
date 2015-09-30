### 简洁

用一行代码创建一个 POJO 同时具有 getters, setters, `equals()`, `hashCode()`, `toString()` 和 `copy()` 方法:

``` kotlin
data class Customer(val name: String, val email: String, val company: String)
```

或者使用lambda表达式过滤一个list:

``` kotlin
val positiveNumbers = list.filter {it > 0}
```

想要一个单例? 创建一个 object 吧:

``` kotlin
object ThisIsASingleton {
  val companyName: String = "JetBrains"
}
```