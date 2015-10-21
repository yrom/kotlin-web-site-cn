### 安全

彻底告别那些烦人的 NullPointerExceptions, [毕竟价值万亿](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)

``` kotlin
var output : String
output = null
```

And of course, Kotlin protects you from mistakenly operating on nullable types,
including those from Java
Kotlin会预防你对可能为null的对象进行操作, 哪怕是Java中传递过来的

``` kotlin
println(output.length())
```

如果你进行了类型检测, 那么编译器会在检测后的代码分支内自动帮你转换类型

``` kotlin
fun calculateTotal(obj: Any) {
  if (obj is Invoice) {
    obj.calculateTotal()
  }
}
```
