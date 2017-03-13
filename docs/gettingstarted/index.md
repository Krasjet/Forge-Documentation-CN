Forge入门
========

这是一个让你了解如何从零开始构建一个基本mod的简单指南。这个文档剩下的内容都是基于本篇教程出发的。

从零起步制作mod
-----------

1. 从Forge[下载站点][files]获取Forge的源码发布版（即Mdk版本，如果是1.8/1.7的旧版本则是Src）。
2. 解压刚下载的源码到一个空文件夹中。你应该能看见有一些文件在里面，我们在 `src/main/java` 中准备了一个范例mod供您参考。只有下面这几个文件是在mod开发中必须的，你可以在你所有的工程中重复使用这些文件：
    - `build.gradle`
    - `gradlew` (`.bat`和`.sh`)
    - `gradle` 文件夹
3. 将上述文件复制到一个新的文件夹中，它将会是你的mod工程文件夹。
4. 在步骤(3)创建的文件夹中打开命令提示符，运行 `gradlew setupDecompWorkspace` (译注: 如果在Linux系统下替换 `gradlew` 为 `./gradlew`，需要自己添加运行权限)。这个指令会从互联网上下载很多的文件，这些文件会用来反编译和构建Minecraft和Forge。由于它会下载一些东西并且反编译Minecraft，这也许会需要很长时间。
5. 选择你的IDE: Forge官方支持使用Eclipse或者是IntelliJ环境进行开发，但你可以使用任何开发环境，不论是NetBeans还是vi/emacs，都可以正常工作
    - 对于Eclipse用户，你需要运行 `gradlew eclipse` - 这会下载更多文件以能够让工程在Eclipse中构建，并且将Eclipse工程输出到你当前的目录
    - 对于IntelliJ用户，直接导入build.gradle文件就可以了（译注：IDEA启动界面Import Project选build.gradle）
6. 加载你的工程到IDE
    - 对于Eclipse用户，在任意地方创建一个工作空间(Workspace)（当然最方便的就是在工程文件夹的上一级目录中创建）。之后以工程的形式导入你的工程文件夹，之后的事情软件都会自动处理
    - 对于IntelliJ用户，你只需要创建运行配置就行了。你可以运行 `gradlew genIntellijRuns` 来自动生成

!!! note

	如果你在运行第4步时看到在 `:decompileMC` 这个任务报错

    ```
    Execution failed for task ':decompileMc'.
    GC overhead limit exceeded
    ```

    请分配更多的内存给Gradle，在 `~/.gradle/gradle.properties` 文件中（如果没有请创建一个）加入 `org.gradle.jvmargs=-Xmx2G` 参数。`~` 符号代表用户的[Home目录][home directory]。

!!! note

    注意，通常情况下，`gradlew setupDecompWorkspace` 的文件只需要被下载并且反编译一次，除非你删除了Gradle的产物缓存。

无需控制台的IntelliJ IDEA配置
----------------------------

在开始这部分之前，请按照步骤1到3先将工程文件夹创建好。也正因为这个原因，这一部分的序号将从4开始。

!!! note "译注"

	国内用户如果使用代理可以在导入工程的时候在Global Gradle settings的Gradle VM options中输入 `-Dhttp.proxyHost=[ip] -Dhttp.proxyPort=[port] -Dhttps.proxyHost=[ip] -Dhttps.proxyPort=[port]` 等参数来设定。在运行Gradle任务的时候修改运行配置文件，在VM options中填入上述参数。

4. 启动IDEA，并选择打开(Open)或导入(Import) `build.gradle` 文件，使用默认的Gradle Wrapper设置。当这个步骤完成之后，你可以打开右边的Gradle面板，当导入完成后里面将会有所有的Gradle任务(Task)。
5. 运行 `setupDecompWorkspace` 这个任务（在 `forgegradle` 任务组中）。这应该会需要一点时间，并且会占用很大的内存。如果失败的话，在IDEA的Gradle设置窗口的 `Gradle VM options` 中添加 `-Xmx3G`，或者也可以修改你的全局Gradle配置。
6. 这个配置任务完成后，你需要运行 `genIntellijRuns` 这个任务，它将会配置工程的运行/调试对象。
7. 完成后，你需要点击**Gradle面板中的**蓝色刷新按钮（在工具栏上也有一个刷新按钮，但不是那个）。这将会重新同步IDEA工程与Gradle数据，保证所有的依赖项与配置都是最新的。
8. 最后，如果你用的是IDEA 2016或者更新版本，你需要修复类路径模块。进入 `Edit configurations`，在 `Minecraft Client` 和 `Minecraft Server` 中，将 `Use classpath of module` 指向类似于 `<project>_main` 这样名字的模块。

如果所有步骤都没有问题，你现在应该可以从下拉列表中选择Minecraft的运行任务，接下来点击Run/Debug按钮来测试你的配置。

自定义你的mod信息
---------------

修改 `build.gradle` 文件从而自定义你的Mod的构建（文件名，版本或者是其它东西）。

!!! important

    **不要**修改build.gradle文件里的 `buildscript {}` 部分，默认的代码对ForgeGradle的运行至关重要。

在 `apply project: forge` 和 `// EDITS GO BELOW HERE` 下面的几乎任何东西都可以被修改，许多东西都可以被删除并且自定义修改。

这里有一个站点来介绍Forge的 `build.gradle` 文件 - [ForgeGradle cookbook][] ([中文版](http://forgegradle-cn.readthedocs.org/zh/latest/))。 一旦你熟悉你mod的设置，你会发现那里很多有用的配方。

[forgegradle cookbook]: https://forgegradle.readthedocs.org/en/latest/cookbook/ "The ForgeGradle cookbook"

### 简单的 `build.gradle` 自定义设置

这些自定义设置是非常推荐所有工程都应用的。

- 改变构建文件的文件名 - 修改 `archivesBaseName` 的值
- 改变你的“Maven坐标” - 修改 `group` 的值
- 改变版本号 - 修改 `version` 的值

构建并测试你的mod
---------------

1. 如果你想构建你的mod，运行 `gradlew build`。这将会输出一个文件到 `build/libs` 目录，它的名字是 `[archivesBaseName]-[version].jar`。这个文件可以放到一个装有Forge的Minecraft的 `mods` 文件夹，并且可以发布出去。
2. 如果你想测试你的mod，最简单的方法是使用在配置工程时生成的运行配置。或者也可以运行 `gradlew runClient`。这将会从 `<runDir>` 位置启动Minecraft，以及你的mod代码。当然，这个指令也有不同的自定义设置。请在 [ForgeGradle cookbook][]里面找更多的信息。
3. 你也可以通过运行配置启动一个专门的服务器，或者使用 `gradlew runServer` 指令。这将会启动一个带有GUI的Minecraft服务器。。


!!! note

	如果你想让你的mod运行在服务器上，我们始终建议您在专门的服务器环境下测试您的mod。
	
[files]: http://files.minecraftforge.net "Forge文件发布站"
[home directory]: https://en.wikipedia.org/wiki/Home_directory#Default_home_directory_per_operating_system "不同系统中默认的用户Home目录位置"
