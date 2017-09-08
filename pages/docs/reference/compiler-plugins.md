---
type: doc
layout: reference
title: "编译器插件"
---

# 编译器插件

## 全开放编译器插件

Kotlin 有类及其默认为 `final` 的成员，这使得像 Spring AOP 这样需要类为 `open` 的框架和库用起来很不方便。
这个 `all-open` 编译器插件会适配 Kotlin 以满足那些框架的需求，并使用指定的注解标注类而其成员无需显式使用 `open` 关键字打开。
例如，当你使用 Spring 时，你不需要打开所有的类，而只需要使用特定的注解标注，如
`@Configuration` 或 `@Service`。
`all-open` 插件允许指定这些注解。

我们为全开放插件提供 Gradle 和 Maven 以及 IDE 集成的支持。
对于 Spring，你可以使用 `kotlin-spring` 编译器插件（[见下文](compiler-plugins.html#kotlin-spring-编译器插件)）。

### 如何使用全开放插件

在 `build.gradle` 中添加插件：

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version"
    }
}

apply plugin: "kotlin-allopen"
```
或者，如果你使用 Gradle 插件 DSL，将其添加到 `plugins` 块：

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.allopen" version "1.1.4-3"
}
```

然后指定会打开该类的注解：

```groovy
allOpen {
    annotation("com.my.Annotation")
}
```

如果类（或任何其超类）标有 `com.my.Annotation` 注解，类本身及其所有成员会变为开放。

它也适用于元注解：

``` kotlin
@com.my.Annotation
annotation class MyFrameworkAnnotation

@MyFrameworkAnnotation
class MyClass // 将会全开放
```

`MyFrameworkAnnotation` 也是使类打开的注解，因为它标有 `com.my.Annotation` 注解。

下面是全开放与 Maven 一起使用的用法：

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <configuration>
        <compilerPlugins>
            <!-- 或者 "spring" 对于 Spring 支持 -->
            <plugin>all-open</plugin>
        </compilerPlugins>

        <pluginOptions>
            <!-- 每个注解都放在其自己的行上 -->
            <option>all-open:annotation=com.my.Annotation</option>
            <option>all-open:annotation=com.their.AnotherAnnotation</option>
        </pluginOptions>
    </configuration>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-allopen</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
    </dependencies>
</plugin>
```


### Kotlin-spring 编译器插件
 
你无需手动指定 Spring 注解，你可以使用 `kotlin-spring` 插件，它根据 Spring 的要求自动配置全开放插件。

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-allopen:$kotlin_version"
    }
}

apply plugin: "kotlin-spring"
```

或者使用 Gradle 插件 DSL：

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.spring" version "1.1.4-3"
}
```

其 Maven 示例与上面的类似。

该插件指定了以下注解：
[`@Component`](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/stereotype/Component.html)、 
[`@Async`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/scheduling/annotation/Async.html)、 
[`@Transactional`](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html)、 
[`@Cacheable`](http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/cache/annotation/Cacheable.html)。
由于元注解的支持，标注有 `@Configuration`、 `@Controller`、 `@RestController`、 `@Service` 或者 `@Repository` 的类会自动打开，因为这些注解标注有元注解 `@Component`。
 
当然，你可以在同一个项目中同时使用 `kotlin-allopen` 和 `kotlin-spring`。
请注意，如果你使用 [start.spring.io](http://start.spring.io/#!language=kotlin)，`kotlin-spring` 插件将默认启用。


## 无参编译器插件

无参（no-arg）编译器插件为具有特定注解的类生成一个额外的零参数构造函数。
这个生成的构造函数是合成的，因此不能从 Java 或 Kotlin 中直接调用，但可以使用反射调用。
这允许 Java Persistence API（JPA）实例化 `data` 类，虽然它从 Kotlin 或 Java 的角度看没有无参构造函数（参见[下面](compiler-plugins.html#kotlin-jpa-编译器插件)的 `kotlin-jpa` 插件的描述）。
 
### 如何使用无参插件

其用法非常类似于全开放插件。
添加该插件并指定注解的列表，这些注解一定会导致被标注的类生成无参构造函数。

在 Gradle 中使用无参插件方法：

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-noarg:$kotlin_version"
    }
}

apply plugin: "kotlin-noarg"
```

或者使用 Gradle 插件 DSL：

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.noarg" version "1.1.4-3"
}
```

然后指定注解类型：

```groovy
noArg {
    annotation("com.my.Annotation")
}
```

如果你希望该插件在合成的构造函数中运行其初始化逻辑，请启用 `invokeInitializers` 选项。由于在未来会解决的 [`KT-18667`](https://youtrack.jetbrains.com/issue/KT-18667) 及 [`KT-18668`](https://youtrack.jetbrains.com/issue/KT-18668)，自 Kotlin 1.1.3-2 起，它被默认禁用。

```groovy
noArg {
    invokeInitializers = true
}
```

在 Maven 中使用无参插件方法：

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <configuration>
        <compilerPlugins>
            <!-- 或者 "jpa" 对于 JPA 支持 -->
            <plugin>no-arg</plugin>
        </compilerPlugins>

        <pluginOptions>
            <option>no-arg:annotation=com.my.Annotation</option>
            <!-- 在合成的构造函数中调用实例初始化器 -->
            <!-- <option>no-arg:invokeInitializers=true</option> -->
        </pluginOptions>
    </configuration>

    <dependencies>
        <dependency>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-noarg</artifactId>
            <version>${kotlin.version}</version>
        </dependency>
    </dependencies>
</plugin>
```

### Kotlin-jpa 编译器插件

该插件指定
[`@Entity`](http://docs.oracle.com/javaee/7/api/javax/persistence/Entity.html) 
和 [`@Embeddable`](http://docs.oracle.com/javaee/7/api/javax/persistence/Embeddable.html) 
注解作为应该为一个类生成无参构造函数的标记。
这就是如何在 Gradle 中添加该插件的方法：

``` groovy
buildscript {
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-noarg:$kotlin_version"
    }
}

apply plugin: "kotlin-jpa"
```

或者使用 Gradle 插件 DSL：

```groovy
plugins {
  id "org.jetbrains.kotlin.plugin.jpa" version "1.1.4-3"
}
```

其 Maven 示例与上面的类似。
