---
type: tutorial
layout: tutorial
title: "Getting started with Android and Kotlin"
description: "This tutorial walks us through creating a simple Kotlin application for Android using Android Studio."
authors: 
showAuthorInfo: false
---

### 安装Kotlin插件

Android Studio 3.0(preview)版本开始将内置安装Kotlin插件。如果您正在使用的是早期版本，需要通过 _File \| Settings \| Plugins \| Install JetBrains plugin..._ 搜索并安装 *Kotlin* 插件。在Android Studio欢迎界面中，可以通过 _Configure \| Plugins \| Install JetBrains plugin..._ 。安装完成后需要重新启动Android Studio。

### 创建新工程

使用Kotlin来进行Android的开发是极其之容易的。在本章教程中，我们直接根据Android Studio的引导进行操作。若您当前使用的IDE是Intellij IDEA，所有操作几乎一致。

首先，创建新的工程。选择 **Start a new Android Studio project** 或者 **File | New project**。根据弹出的对话框提示进行操作。为新工程命名，选择已安装的SDK。实际上大多数选项均有默认值 ，只需要按几次“回车”键即可。

命名工程:
![Dialog 1]({{ url_for('tutorial_img', filename='kotlin-android/0-create-new-project.png') }})

Android Studio 3.0 在当前对话框中提供Kotlin支持的选项，勾选后可以跳过"配置Kotlin工程(Configuring Kotlin in the project)"的步骤。

选择Android版本:

![Dialog 2]({{ url_for('tutorial_img', filename='kotlin-android/1-create-new-project.png') }})

选择需要创建的Activity样式:

![Dialog 3]({{ url_for('tutorial_img', filename='kotlin-android/2-create-new-project.png') }})

命名Activity:

![Dialog 4]({{ url_for('tutorial_img', filename='kotlin-android/3-create-new-project.png') }})

在 Android Studio 3.0中，可以选择使用Kotlin创建activity，因此也不需要"将Java代码转换为Kotlin(Converting
Java code to Kotlin)"这一步骤。早期版本中则会先使用Java创建activity，然后再使用自动转换工具进行操作。

一般而言，着手使用Kotlin的最便捷方式，无疑是将Java代码自动转换为Kotlin。值得一提的是，与其为了使用新的方式表达旧的模式而去查阅文档，不如直接使用Java编写代码，再复制粘贴到Kotlin文件，IntelliJ IDEA(或Android Studio)会提示需要转换代码。
#### Java代码转换Kotlin

打开 `MainActivity.java` 文件，使用 **Convert Java File to Kotlin File** 命令。使用该命令的方式有如下几种，强烈推荐[Find Action](https://www.jetbrains.com/idea/help/navigating-to-action.html)，输入相应的命令名称（见下方截图）即可。同样也可以通过菜单栏依次调出 _Code \| Convert Java File to Kotlin File_ 或使用快捷键(菜单栏入口可见)完成操作。
 
![Convert]({{ url_for('tutorial_img', filename='kotlin-android/convert-java-to-kotlin.png') }})

转换完成后即可看到使用Kotlin编写的activity。

![Koltin-Activity]({{ url_for('tutorial_img', filename='kotlin-android/converted-code.png') }})

#### 配置Kotlin工程

在开始编辑此文件时，Android Studio会提示当前工程还未配置Kotlin，根据提示完成操作即可；相应的也可以在菜单栏中选择 Tools | Kotlin | Configure Kotlin 完成配置。  

![Config-Kotlin]({{ url_for('tutorial_img', filename='kotlin-android/kotlin-not-configured.png') }})

选择配置时有如下对话框，选择已安装的最新版本即可。

![Config-Kotlin-Details]({{ url_for('tutorial_img', filename='kotlin-android/configure-kotlin-in-project-details.png') }})

Kotlin配置完成后，应用程序的build.gradle文件会新增 _apply plugin: 'kotlin-android'_ 的依赖。

*(有关使用gradle设置的详情，请查阅[Gradle使用手册](/docs/reference/using-gradle.html))*
 
![Sync-Project-With-Gradle]({{ url_for('tutorial_img', filename='kotlin-android/sync-project-with-gradle.png') }})

同步工程，在提示框中点击'立即同步(Sync Now)'或者使用**Sync Project with Gradle Files**即可。

![Sync-Project-With-Gradle-2]({{ url_for('tutorial_img', filename='kotlin-android/sync-project-with-gradle-2.png') }})

### 构建和发布Android的Kotlin应用程序

最后构建应用程序，并发布到虚拟机或连接的设备；所有这些操作和Java并无区别，您可以发布应用程序，并以类似于使用Java编写的Android应用程序的方式进行签名。

Kotlin有着极小的运行时文件体积：整个库的大小约{{ site.data.releases.latest.runtime_size }} ( {{ site.data.releases.latest.version }} 版本)。这意味着Kotlin对apk文件大小影响微乎其微。

而对于Kotlin编译器生成的字节码，对于Kotlin与Java所编写的程序而言，丝毫没有任何差异。

### 后续?

- 更多内容请查阅[Kotlin Android Extensions plugin](android-plugin.html) 以及 [Android Frameworks Using Annotation Processing](android-frameworks.html)。
- 想尝试Kotlin的不同特性？试试 [Kotlin Koans](koans.html)。
- 查看[Google的Kotlin示例工程](https://developer.android.com/samples/index.html?language=kotlin)