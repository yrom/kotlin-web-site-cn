---

type: doc
layout: reference
title: "使用 Maven"
description: "This tutorials walks you through different scenarios when using Maven for building applications that contain Kotlin code"

---

# 使用 Maven

## 插件与版本

*kotlin-maven-plugin* 用于编译 Kotlin 源码与模块，当前只支持 Marven V3

通过 *kotlin.version* 指定所要使用的 Kotlin 版本，The correspondence between Kotlin releases and versions is displayed below:

<table>
<thead>
<tr>
  <th>Milestone</th>
  <th>Version</th>
</tr>
</thead>
<tbody>
{% for entry in site.data.releases.list %}
<tr>
  <td>{{ entry.milestone }}</td>
  <td>{{ entry.version }}</td>
</tr>
{% endfor %}
</tbody>
</table>


## 依赖

Kotlin 提供了大量的标准库以供开发使用，需要在 pom 文件中设置以下依赖：

``` xml
<dependencies>
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-stdlib</artifactId>
        <version>${kotlin.version}</version>
    </dependency>
</dependencies>
```

## 仅编译 Kotlin 源码

在 &lt;build&gt; 标签中指定所要编译的 Kotlin 源码目录：

``` xml
<sourceDirectory>${project.basedir}/src/main/kotlin</sourceDirectory>
<testSourceDirectory>${project.basedir}/src/test/kotlin</testSourceDirectory>
```

Maven 中需要引用 Kotlin 插件用于编码源码：

``` xml

<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <executions>
        <execution>
            <id>compile</id>
            <goals> <goal>compile</goal> </goals>
        </execution>

        <execution>
            <id>test-compile</id>
            <goals> <goal>test-compile</goal> </goals>
        </execution>
    </executions>
</plugin>
```

## 同时编译 Kotlin 与 Java 源码

编译混合代码时 Kotlin 编译器应先于 Java 的编译器被调用。
在 Maven 中这表示 kotlin-maven-plugin 先于 maven-compiler-plugin 运行。

It could be done by moving Kotlin compilation to previous phase, process-sources（如果有更好的解决方案欢迎提出）：

``` xml
<plugin>
    <artifactId>kotlin-maven-plugin</artifactId>
    <groupId>org.jetbrains.kotlin</groupId>
    <version>${kotlin.version}</version>

    <executions>
        <execution>
            <id>compile</id>
            <phase>process-sources</phase>
            <goals> <goal>compile</goal> </goals>
        </execution>

        <execution>
            <id>test-compile</id>
            <phase>process-test-sources</phase>
            <goals> <goal>test-compile</goal> </goals>
        </execution>
    </executions>
</plugin>
```

## OSGi

OSGi支持查看 [Kotlin OSGi page](kotlin-osgi.html).

## 例子

Maven 工程的例子可从 [Github 直接下载](https://github.com/JetBrains/kotlin-examples/archive/master/maven.zip)


