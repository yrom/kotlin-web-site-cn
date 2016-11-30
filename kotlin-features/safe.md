### 安全

彻底告别那些烦人的 NullPointerException，毕竟[价值万亿](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)。

``` kotlin
var output : String
output = null
```

当然，Kotlin 会保护你避免操作可控类型，
包括来自 Java 的

``` kotlin
println(output.length())
```

并且如果你检查类型是正确的，编译器会为你做自动类型转换

``` kotlin
fun calculateTotal(obj: Any) {
    if (obj is Invoice) {
        obj.calculateTotal()
    }
}
```
