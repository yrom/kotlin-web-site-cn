### 安全

避免那些令人讨厌的空指针异常，你知道的，这可能酿成数十亿美元的损失

``` kotlin
var output : String
output = null
```

当然Kotlin让你避免那些错误通过使用nullable类型，
也包括Java的

``` kotlin
println(output.length())
```

在你进行类型检查的时候，如果类型匹配，那么编译器会帮你进行自动转换

``` kotlin
fun calculateTotal(obj: Any) {
  if (obj is Invoice) {
    obj.calculateTotal()
  }
}
```