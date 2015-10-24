Forge入门
==========================

这是一个让你了解如何从零到构建一个基本mod的简单指南. 这个文档剩下的内容是关于从这里出发该何去何从.

从零到制作mod
--------------------

1. 从Forge[下载站点][files]获取Forge的源码发布版(即Mdk版本(旧版本为Src)).

2. 解压下载到的源码到一个空文件夹. 你应该看见有一些文件在里面，有一个范例mod在 `src/main/java` 中供您参考.

3. 运行 `gradlew setupDecompWorkspace`(译注: 如果在Linux系统下替换`gradlew`为`./gradlew`，自己添加运行权限). 这个指令会从互联网上下载很多的文件，这些文件会用来反编译和构建Minecraft和Forge. 这也许会需要很长时间，因为它会下载一些东西并且反编译Minecraft.

4. 选择你的IDE: Forge支持使用Eclipse或者是IntelliJ环境进行开发(虽然你也可以自己导入产物到你的环境当中并且任意修改mod代码 - 从`cat >> mymodfile`到vi到emacs都可以).
    * 对于Eclipse用户，你应该运行 `gradlew eclipse` - 这会下载更多为了构建Eclipse工程的产物，并且将Eclipse工程输出到你当先的目录.
    * 对于IntelliJ用户，你应该运行 `gradlew idea` - 这会下载为了构建IntelliJ IDEA工程的产物，并将IntelliJ工程输出到你当前的目录.
5. 加载你的工程到IDE.
    * 对于Eclipse用户，使用 `Import | Existing Projects into Workspace` 之后选中你之前解压的目录
    * 对于IntelliJ用户,使用 `File | Open` 并且打开你解压目录下的.ipr文件.
6. 修改示例代码，或者导入你自己已有的mod代码，或者创建你自己的新mod.

!!! note

    注意，通常情况下，`gradlew setupDecompWorkspace` 的文件只需要被下载并且反编译一次，除非你删除了Gradle产物缓存.

自定义你的mod信息
--------------------------------

修改`build.gradle`文件从而自定义你的mod如何能够构建(文件名，版本或者是其他东西).

!!! important

    **不要**修改你的build.gradle文件里的 `buildscript {}` 部分 - 他们是特别的.

在 `apply project: forge` 和 `// EDITS GO BELOW HERE` 下面的几乎任何东西都可以被修改，许多东西都可以被删除并且自定义修改.

这里有一整个站点来介绍Forge的`build.gradle` 文件 - [ForgeGradle cookbook][]([中文版](http://forgegradle-cn.readthedocs.org/zh/latest/)，正在翻译中). 一旦你熟悉你mod的设置，你会发现那里很多有用的配方.

[forgegradle cookbook]: https://forgegradle.readthedocs.org/en/latest/cookbook/ "The ForgeGradle cookbook"

### 简单的 `build.gradle` 自定义设置

这些自定义设置是非常推荐所有工程都应用的.

* 改变你构建的文件名 - 修改 `archivesBaseName` 的值.
* 改变你的Maven坐标 - 修改 `group` 的值
* 改变版本号 - 修改 `version` 的值.

构建和测试你的mod
-----------------------------

1. 如果你想构建你的mod，运行`gradlew build`. 这将会输出一个文件到 `build/libs` 目录，它的名字是 `<archivesBaseName>-<version>.jar`. 这个文件可以放到一个装有Forge的Minecraft的 `mods` 文件夹，并且可以发布出去.
2. 如果你想测试你的mod，运行 `gradlew runClient`. 这将会从 `<runDir>` 位置启动Minecraft，包括你的mod代码. 这里也有对于这个指令不同的自定义设置. 请在 [ForgeGradle cookbook][]里面找更多的信息.
3. 你也可以启动一个专门的服务器，使用 `gradlew runServer` 指令. 这将会启动一个带有GUI的Minecraft服务器. **如果你想让你的mod运行在服务器上，我们始终建议您在专门的服务器环境下测试您的mod**.
4. 你也可以在您的IDE环境下运行Minecraft:...

[files]: http://files.minecraftforge.net "Forge文件发布"
