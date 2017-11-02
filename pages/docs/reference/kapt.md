---
type: doc
layout: reference
title: "使用 kapt"
---

# Kotlin 注解处理

> 译注：kapt 即 Kotlin annotation processing tool（Kotlin 注解处理工具）缩写。

Annotation processors (see [JSR 269](https://jcp.org/en/jsr/detail?id=269)) are supported in Kotlin with the *kapt* compiler plugin.

Being short, you can use libraries such as [Dagger](https://google.github.io/dagger/) or [Data Binding](https://developer.android.com/topic/libraries/data-binding/index.html) in your Kotlin projects.

Please read below about how to apply the *kapt* plugin to your Gradle/Maven build.

## Using in Gradle

Apply the `kotlin-kapt` Gradle plugin:

```groovy
apply plugin: 'kotlin-kapt'
```

或者，你可以使用插件 DSL 应用它：

```groovy
plugins {
    id "org.jetbrains.kotlin.kapt" version "{{ site.data.releases.latest.version }}"
}
```

然后在 `dependencies` 块中使用 `kapt` 配置添加相应的依赖项：

``` groovy
dependencies {
    kapt 'groupId:artifactId:版本'
}
```

如果你以前使用 [Android support](https://developer.android.com/studio/build/gradle-plugin-3-0-0-migration.html#annotationProcessor_config) 作为注解处理器，那么以 `kapt` 取代 `annotationProcessor` 配置的使用。如果你的项目包含 Java 类，`kapt` 也会顾全到它们。

如果为 `androidTest` 或 `test` 源代码使用注解处理器，那么相应的 `kapt` 配置名为 `kaptAndroidTest` 和 `kaptTest`。请注意 `kaptAndroidTest` 和 `kaptTest` 扩展了 `kapt`，所以你可以只提供 `kapt` 依赖而它对生产和测试源代码都可用。

一些注解处理器（如 `AutoFactory`）依赖于声明签名中的精确类型。默认情况下，Kapt 将每个未知类型（包括生成的类的类型）替换为 `NonExistentClass`，但你可以更改此行为。将额外标志添加到 `build.gradle` 文件以启用在存根（stub）中推断出的错误类型：

``` groovy
kapt {
    correctErrorTypes = true
}
```

## Using in Maven

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


## Using in CLI

Kapt compiler plugin is available in the binary distribution of the Kotlin compiler.

You can attach the plugin by providing the path to its JAR file using the `Xplugin` kotlinc option:

```bash
-Xplugin=$KOTLIN_HOME/lib/kotlin-annotation-processing.jar
```

Here is a list of the available options:

* `sources` (*required*): An output path for the generated files.
* `classes` (*required*): An output path for the generated class files and resources.
* `stubs` (*required*): An output path for the stub files. In other words, some temporary directory.
* `incrementalData`: An output path for the binary stubs.
* `apclasspath` (*repeatable*): A path to the annotation processor JAR. Pass as many `apclasspath` options as many JARs you have.
* `apoptions`: A base64-encoded list of the annotation processor options. See [AP/javac options encoding](#apjavac-options-encoding) for more information.
* `javacArguments`: A base64-encoded list of the options passed to javac. See [AP/javac options encoding](#apjavac-options-encoding) for more information.
* `processors`: A comma-specified list of annotation processor qualified class names. If specified, kapt does not try to find annotation processors in `apclasspath`.
* `verbose`: Enable verbose output.
* `aptMode` (*required*)
    * `stubs` – only generate stubs needed for annotation processing;
    * `apt` – only run annotation processing;
    * `stubsAndApt` – generate stubs and run annotation processing.
* `correctErrorTypes`: See [below](#using-in-gradle). Disabled by default.

The plugin option format is: `-P plugin:<plugin id>:<key>=<value>`. Options can be repeated.

An example:

```bash
-P plugin:org.jetbrains.kotlin.kapt3:sources=build/kapt/sources
-P plugin:org.jetbrains.kotlin.kapt3:classes=build/kapt/classes
-P plugin:org.jetbrains.kotlin.kapt3:stubs=build/kapt/stubs

-P plugin:org.jetbrains.kotlin.kapt3:apclasspath=lib/ap.jar
-P plugin:org.jetbrains.kotlin.kapt3:apclasspath=lib/anotherAp.jar

-P plugin:org.jetbrains.kotlin.kapt3:correctErrorTypes=true
```


## Generating Kotlin sources

Kapt can generate Kotlin sources. Just write the generated Kotlin source files to the directory specified by `processingEnv.options["kapt.kotlin.generated"]`, and these files will be compiled together with the main sources.

You can find the complete sample in the [kotlin-examples](https://github.com/JetBrains/kotlin-examples/tree/master/gradle/kotlin-code-generation) Github repository.

Note that Kapt does not support multiple rounds for the generated Kotlin files.


## AP/javac options encoding

`apoptions` and `javacArguments` CLI options accept an encoded map of options.  
Here is how you can encode options by yourself:

```kotlin
fun encodeList(options: Map<String, String>): String {
    val os = ByteArrayOutputStream()
    val oos = ObjectOutputStream(os)

    oos.writeInt(options.size)
    for ((key, value) in options.entries) {
        oos.writeUTF(key)
        oos.writeUTF(value)
    }

    oos.flush()
    return Base64.getEncoder().encodeToString(os.toByteArray())
}
```
