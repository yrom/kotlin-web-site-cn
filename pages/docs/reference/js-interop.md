---
type: doc
layout: reference
category: "JavaScript"
title: "JavaScript 互操作性"
---

# JavaScript 互操作性

## JavaScript 模块

从 Kotlin 1.0.4 版开始，你可以将 Kotlin 项目编译为热门模块系统的 JS 模块。以下是
可用选项的列表：

1. 无模块（Plain）。不为任何模块系统编译。像往常一样，你可以
   通过 `kotlin.modules.moduleName` 访问模块 `moduleName`，或者直接通过 `moduleName` 标识符访问放在全局作用域内的。
   默认使用此选项。
2. [异步模块定义（AMD，Asynchronous Module Definition）](https://github.com/amdjs/amdjs-api/wiki/AMD)，它尤其为
   require.js 库所使用。
3. [CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1) 约定，广泛用于 node.js/npm
   （`require` 函数和 `module.exports` 对象）
4. 统一模块定义（UMD，Unified Module Definitions），它与 *AMD* 和 *CommonJS* 兼容，
   并且当 *AMD* 和 *CommonJS* 都不可用时，作为“plain”使用。

选择目标模块系统的方式取决于你的构建环境：

### 对于 IDEA

打开“File → Settings”，选择“Build, Execution, Deployment”→“Compiler”→“Kotlin compiler”。 在
“Module kind”字段中选择合适的模块系统。


### 对于 Maven

要选择通过 Maven 编译时的模块系统，你应该设置 `moduleKind` 配置属性，即你的
`pom.xml` 应该看起来像这样：

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>
    <executions>
        <execution>
            <id>compile</id>
            <goals>
                <goal>js</goal>
            </goals>
        </execution>
    </executions>
    <!-- 插入这些行 -->
    <configuration>
        <moduleKind>commonjs</moduleKind>
    </configuration>
    <!-- 插入文本结束 -->
</plugin>
```

可用值包括：`plain`、 `amd`、 `commonjs`、 `umd`。


### 对于 Gradle

要选择通过 Gradle 编译时的模块系统，你应该设置 `moduleKind` 属性，即

    compileKotlin2Js.kotlinOptions.moduleKind = "commonjs"

可用的值类似于 Maven


### 注意事项

我们将 `kotlin.js` 标准库作为一个单独的文件，它本身编译为一个 UMD 模块，所以
你可以与上述任何模块系统一起使用。

虽然现在我们没有直接支持 WebPack 和 Browserify，但是我们测试了由 Kotlin 编译器生成的 `.js` 文件
与 WebPack 和 Browserify 一同使用，所以 Kotlin 应该能正确使用这些工具。


## @JsName 注解

在某些情况下（例如，为了支持重载），Kotlin编译器会修饰（mangle）
JavaScript 代码中生成的函数和属性的名称。如果要控制生成的名称，可以使用 `@JsName` 注解：

``` kotlin
// 模块 'kjs'

class Person(val name: String) {
    fun hello() {
        println("Hello $name!")
    }

    @JsName("helloWithGreeting")
    fun hello(greeting: String) {
        println("$greeting $name!")
    }
}
```

现在你可以从 JavaScript 中用以下方式使用该类：

``` javascript
var person = new kjs.Person("Dmitry");   // 引用到模块 'kjs'
person.hello();                          // 输出 "Hello Dmitry!"
person.helloWithGreeting("Servus");      // 输出 "Servus Dmitry!"
```

如果我们没有指定 `@JsName` 注解，相应函数的名称将包含
从函数签名计算而来的后缀，例如 `hello_61zpoe$`。
