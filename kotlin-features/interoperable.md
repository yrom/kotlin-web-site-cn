
### 互操作性

随意创建和使用 Java 代码

``` kotlin
import io.netty.channel.ChannelInboundMessageHandlerAdapter
import io.netty.channel.ChannelHandlerContext

public class NettyHandler: ChannelInboundMessageHandlerAdapter<Any>() {
    public override fun messageReceived(p0: ChannelHandlerContext?, p1: Any?) {
        throw UnsupportedOperationException()
    }
}
```

或者使用任何 JVM 上已有的库，因为百分百兼容，包括 SAM 支持。

无论是 JVM 还是 JavaScript 目标平台，都可用 Kotlin 写代码然后部署到你想要的地方

``` kotlin
import js.dom.html.*

fun onLoad() {
    window.document.body.innerHTML += "<br/>Hello, Kotlin!"
}
```
