
### Interoperable

Create and consume Java code at will

``` kotlin
import io.netty.channel.ChannelInboundMessageHandlerAdapter
import io.netty.channel.ChannelHandlerContext

public class NettyHandler: ChannelInboundMessageHandlerAdapter<Any>() {
    public override fun messageReceived(p0: ChannelHandlerContext?, p1: Any?) {
        throw UnsupportedOperationException()
    }
}
```

或者在JVM上使用已有Java库, 百分百兼容, 包括SAM支持

无论是JVM还是JavaScript, 用Kotlin写代码, 然后部署到你想要的地方.

``` kotlin
import js.dom.html.*

fun onLoad() {
    window.document.body.innerHTML += "<br/>Hello, Kotlin!"
}
```
