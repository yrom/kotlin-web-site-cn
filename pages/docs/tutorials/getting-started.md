---
type: tutorial
layout: tutorial
title:  "以 IntelliJ IDEA 入门"
description: "本教程将引导你使用 IntelliJ IDEA 创建一个简单的 Hello World 应用程序。"
authors: Hadi Hariri, Roman Belov
date: 2017-05-04
showAuthorInfo: false
---
### 环境搭建
在本教程中，我们会使用 IntelliJ IDEA。
关于如何使用命令行编译器编译和执行 Kotlin 应用程序的说明，请参见[使用命令行编译器][getting_started_command_line]。

如果你是 JVM 和 Java 的新手，请查看 [JVM Minimal Survival Guide](http://hadihariri.com/2013/12/29/jvm-minimal-survival-guide-for-the-dotnet-developer/)。如果你是 IntelliJ IDEA 的新手，请查看 [The IntelliJ IDEA Minimal Survival Guide](http://hadihariri.com/2014/01/06/intellij-idea-minimal-survival-guide/)。

1. **安装最新版本的 IntelliJ IDEA**。从版本 15 开始，Kotlin 就已内置于 IntelliJ IDEA 了。
   你可以从 [JetBrains][jetbrains] 下载免费的[社区版][intellijdownload]。

2. 创建一个新项目。我们选择 Java Module 并选择 SDK。 Kotlin 与 JDK 1.6+ 一起使用。 另外，选择 *Kotlin (Java)* 复选框。

   ![新建 Kotlin 项目]({{ url_for('tutorial_img', filename='getting-started/new_project_step1.png') }})

3. 下一步给我们的项目起个名字。

   ![Kotlin 项目名]({{ url_for('tutorial_img', filename='getting-started/project_name.png') }})

4. 我们现在应该已经创建了新项目并有如下文件夹解构：

   ![Kotlin 文件夹解构]({{ url_for('tutorial_img', filename='getting-started/folders.png') }})

5. 让我们在源代码文件夹下创建一个新的 Kotlin 文件。它可以任意命名。在本例中，我们称之为 *app*。

   ![新建 Kotlin 文件]({{ url_for('tutorial_img', filename='getting-started/new_file.png') }})

6. 我们创建完文件后，我们需要键入主程序，这是 Kotlin 应用程序的入口点。IntelliJ IDEA 为我们提供了一个快速完成此操作的模板。只需键入 *main* 然后按 Tab 即可。

   ![Kotlin Main 函数]({{ url_for('tutorial_img', filename='getting-started/main.png') }})

7. 现在我们添加一行代码来打印出“Hello, World!”吧。

   ![Kotlin 添加代码]({{ url_for('tutorial_img', filename='getting-started/hello_world.png') }})

8. 现在我们可以运行应用程序了。最简单的方法是点击左边框中的 Kotlin 图标，然后选择 *Run 'AppKt'*。

   ![Kotlin 运行程序]({{ url_for('tutorial_img', filename='getting-started/run_default.png') }})

9. 如果一切顺利，我们现在会在工具窗口 **Run** 中看到结果。

   ![Kotlin 运行窗口]({{ url_for('tutorial_img', filename='getting-started/run_window.png') }})

恭喜！ 我们现在让我们的第一个应用程序运行了。

[intellijdownload]: http://www.jetbrains.com/idea/download/index.html
[jetbrains]: http://www.jetbrains.com
[getting_started_command_line]: command-line.html
