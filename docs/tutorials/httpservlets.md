---
type: tutorial
layout: tutorial
title:  "使用HttpServlet创建WEB应用"
description: "本教程通过使用HttpServlet创建一个简单的控制器来显示 Hello World。"
authors: biezhi
showAuthorInfo: true
date: 2015-12-30
source: servlet-web-applications
---
JavaEE的HttpServlet可以使用Kotlin，就像使用其他的Java库或者框架一样。我们将看到如何让一个简单的控制器返回 "Hello, World!"

### 定义项目和依赖关系

在本教程中我们将使用Gradle进行构建，但同样可以实现使用IntelliJ IDEA项目结构或Maven的效果。关于如何使用Gradle配置Kotlin，请看[使用Gradle](http://tanfujun.com/docs/reference/using-gradle.html)。HTTP servlet必须依赖 JAVAEE API:

``` groovy
dependencies {
    compile group: 'javax', name: 'javaee-api', version: '7.0'

    testCompile group: 'junit', name: 'junit', version: '4.11'
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}
```

我们还需要 *war* 插件，帮助我们生成相应的构件运行和部署。

``` groovy
apply plugin: war
```

可以在Github检出项目来查看完整的Gradle脚本。

### 创建一个HomeController

一旦我们在构建脚本中定义了正确的依赖，现在就可以创建一个控制器

``` kotlin
@WebServlet(name = "Hello", value = "/hello")
class HomeController : HttpServlet() {
    override fun doGet(req: HttpServletRequest, res: HttpServletResponse) {
        res.writer.write("Hello, World!")
    }
}
```

### 运行程序

使用IntelliJ IDEA，我们可以很容易地运行和调试应用程序，在任何可能的应用程序服务器定义 如Tomcat，Glassfish或WildFly。在这种情况下我们要使用Tomcat[曾被定义为一个应用程序服务器在IntelliJ IDEA](http://www.jetbrains.com/idea/webhelp/defining-application-servers-in-intellij-idea.html)

为了运行，我们需要相应的 WAR(s) 部署。我们可以使用 *war* 生成这些任务，可以很容易地通过在IntelliJ IDEA的Gradle工具执行。

![Gradle Tasks](http://kotlinlang.org/assets/images/tutorials/httpservlets/gradle-tasks.png)

或者使用命令行构建:

```sh
gradle war
```

下一步是在IntelliJ IDEA中创建一个运行配置，使用 Tomcat/Local 部署并启动 WAR。

![Run Config](http://kotlinlang.org/assets/images/tutorials/httpservlets/tomcat-config.png)

一旦我们运行应用程序(使用之前的配置)，成功部署，应该能够使用浏览器打开URL看到如下的响应:

![Browser Run](http://kotlinlang.org/assets/images/tutorials/httpservlets/browser.png)
