---
type: doc
layout: reference
title: "使用 kapt"
---

# 使用 Kotlin 注解处理工具

> 译注：kapt 即 Kotlin annotation processing tool（Kotlin 注解处理工具）缩写。

Kotlin 插件支持注解处理器，如 _Dagger_ 或 _DBFlow_ 。
为了让它们与 Kotlin 类一起工作，需要应用 `kotlin-kapt` 插件：

## Gradle 配置

``` groovy
apply plugin: 'kotlin-kapt'
```

或者，从 Kotlin 1.1.1 起，你可以使用插件 DSL 应用它：

``` groovy
plugins {
    id "org.jetbrains.kotlin.kapt" version "<version to use>"
}
```

然后在 `dependencies` 块中使用 `kapt` 配置添加相应的依赖项：

``` groovy
dependencies {
    kapt 'groupId:artifactId:version'
}
```

如果你以前使用 [android-apt](https://bitbucket.org/hvisser/android-apt) 插件，请将其从 `build.gradle` 文件中删除，并用 `kapt` 取代 `apt` 配置的用法。如果你的项目包含 Java 类，`kapt` 也会顾全到它们。

如果为 `androidTest` 或 `test` 源代码使用注解处理器，那么相应的 `kapt` 配置名为 `kaptAndroidTest` 和 `kaptTest`。请注意 `kaptAndroidTest` 和 `kaptTest` 扩展了 `kapt`，所以你可以只提供 `kapt` 依赖而它对生产和测试源代码都可用。

一些注解处理器（如 `AutoFactory`）依赖于声明签名中的精确类型。默认情况下，Kapt 将每个未知类型（包括生成的类的类型）替换为 `NonExistentClass`，但你可以更改此行为。将额外标志添加到 `build.gradle` 文件以启用在存根（stub）中推断出的错误类型：

``` groovy
kapt {
    correctErrorTypes = true
}
```

请注意，这个选项是实验性的，且默认是禁用的。


## Maven 配置（自 Kotlin 1.1.2 起）

在 `compile` 之前在 kotlin-maven-plugin 中添加 `kapt` 目标的执行：

```xml
<execution>
    <id>kapt</id>
    <goals>
        <goal>kapt</goal>
    </goals>
    <configuration>
        <sourceDirs>
            <sourceDir>src/main/kotlin</sourceDir>
            <sourceDir>src/main/java</sourceDir>
        </sourceDirs>
        <annotationProcessorPaths>
            <!-- 在此处指定你的注解处理器。 -->
            <annotationProcessorPath>
                <groupId>com.google.dagger</groupId>
                <artifactId>dagger-compiler</artifactId>
                <version>2.9</version>
            </annotationProcessorPath>
        </annotationProcessorPaths>
    </configuration>
</execution>
```
 
你可以在
[Kotlin 示例版本库](https://github.com/JetBrains/kotlin-examples/tree/master/maven/dagger-maven-example) 中找到一个显示使用 Kotlin、Maven 和 Dagger 的完整示例项目。
 
请注意，IntelliJ IDEA 自身的构建系统目前还不支持 kapt。当你想要重新运行注解处理时，请从“Maven Projects”工具栏启动构建。
