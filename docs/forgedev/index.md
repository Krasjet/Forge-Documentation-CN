入门
====

如果你想参与Forge的开发的话，在开始写代码之前你还需要进行一些特殊的步骤。普通的Mod开发环境是不能直接对Forge代码进行开发的。这篇指南将告诉你该如何配置环境，以及如何对Forge进行改进！

Fork并Clone仓库
---------------

和大部分开源项目一样，Forge也是托管在[GitHub](https://www.github.com)上的。如果你曾经对其它的工程贡献了代码，那么你应该对这一部分很熟悉了，你可以直接跳到下一个部分中。

如果你是使用Git的新手，那么下面这两个步骤能帮助你快速入门。

!!! note

	这篇指南假设你已经注册了一个GitHub账号。如果你没有的话，访问它的[注册页面](https://www.github.com/join)来创建一个账号。另外，这篇指南并不是Git的使用教程，如果你不能让其正常工作的话，请查阅其它的资源。

### Fork

首先，你需要Fork一下[MinecraftForge的仓库](https://www.github.com/MinecraftForge/MinecraftForge)(Repository)，点击页面右上角的“Fork”按钮。如果你是一个组织(Organization)，请选择一个需要Fork进的账号。

Fork这个仓库是必要的步骤，因为并不是所有的GitHub用户对每个仓库拥有所有的权限的。你需要创建原仓库的一个拷贝，之后通过所谓的Pull Request来提交你的改动，我们将在后面再讨论它。

### Clone

在Fork这个仓库之后，我们需要将代码下载到本地，并做出一些修改。你需要将这个仓库Clone到本地机器上。

使用你最喜欢的Git客户端，将你的Fork Clone到一个自选的目录中。下面这个代码片段应该能在所有配置好Git的系统上工作，它会Clone这个仓库到当前目录中的一个叫做`Forge`的目录中（注意你需要将 `<User` 替换为你的用户名）：

```bash
git clone https://github.com/<User>/MinecraftForge Forge
```

### Check out正确的分支

只有Fork并Clone仓库是Forge开发的必要步骤。但是，为了方便创建Pull Request(PR)，最好是需要分支(Branch)的。

建议是对每个要提交的PR都创建并Check Out到一个新的分支的。这样子，你就能让新的PR保持Forge改动的最新，而仍能够对更老的补丁(Patch)进行开发。

在完成这一步骤之后，你就准备好配置开发环境了。

配置环境
-------

取决于你的IDE，你需要采取不同的推荐步骤来配置一个开发环境。

### Eclipse

由于Eclipse的工作空间(Workspace)的工作方式，ForgeGradle能够帮你做大部分的工作，创建一个Forge工作空间。

1. 打开一个终端/命令提示符，移动到Clone的目录下
2. 输入`./gradlew setupForge`并按下回车。等待ForgeGradle工作完成
3. 打开你的Eclipse工作空间，选择`File -> Import -> General -> Existing Projects into workspace`
4. 在打开的选项框中，将根目录(Root Directory)设置为`<repo>/projects/`
5. 保证“Forge”和“Clean”选项框是勾选状态，并且根据你的喜好调整其它的设置
6. 点击“Finish”按钮完成导入

这就是Eclipse配置的全部过程了，不需要有额外的步骤就能让测试Mod运行。只需要和其它项目一样点击Run按钮并选择所需的运行配置就可以了。

### IntelliJ IDEA


JetBrains的旗舰级IDE对[Gradle](https://www.gradle.org)，Forge所用的构建系统，有着不错的支持。然而，由于Minecraft Mod开发的特殊，我们仍需要进行一些附加的步骤来让它正常工作。

如果你更喜欢视频教程，cpw上传了一个[视频](https://www.youtube.com/watch?v=yanCpy8p2ZE)，里面介绍了非常相似的步骤，也能配置成功。

!!! note

	这些步骤只在IDEA 2016版之后能够稳定工作。更旧的版本没有正式的Gradle支持，而且也不支持Forge的开发工作空间。

1. 将Forge的`build.gradle`文件导入为一个IDEA工程。点击`File -> Open`，之后切换至Clone的目录，选择`build.gradle`文件。如果弹出一个对话框，选择“Open as Project”
2. 在之后的向导中，保证“Create separate module per source set”选项是选中状态，并且“Use default gradle wrapper”也是选中状态。点击确认
3. 在IDEA导入工程和索引文件完成之后，打开屏幕右边的Gradle的侧边栏
4. 打开“forge”工程树，选择“Tasks”，之后是“forgegradle”，之后右击“Create Forge [setup]”选项
5. 在配置对话框打开之后，修改“tasks”字段，写入“clean setup”，并将`-Xmx3G -Xms3G`添加到“VM Options”中。后面那个选项保证了资源开销很大的反编译过程能有足够的内存
6. 点击“Okay”，并运行刚创建的运行配置。这可能会需要运行一段时间。
7. 在配置任务完成之后，再次打开Gradle的侧边栏，点击顶部的“Attach Gradle project”按钮（那个加号图标）
8. 切换至Clone的目录，打开`projects`目录，双击里面的`build.gradle`文件。在之后的对话框中选择“Use gradle wrapper task configuration”，并确认
9. 导入IDEA所有建议的模组
10. 要想使用工程的运行配置，在文件浏览器中打开`projects`目录，进入`.idea`目录（根据系统不同，这个目录可能是隐藏的）。复制`runConfigurations`目录到Clone根目录的`.idea`目录中
11. 当IDEA识别到添加的配置后，对每个配置都执行以下的步骤
	- 将配置的模组改为`<Config>_main`，其中`<Config>`就是配置名称的第一部分
	- 将运行目录改为`<clone>/projects/run`

这就是在IntelliJ IDEA中创建Forge开发环境的全部步骤了。然而，你现在还不能直接运行测试和Forge自带的调试Mod。这还需要一些附加的配置。

#### 启用测试Mod

要想启用Forge自带的测试Mod，你需要将编译器输出添加到Classpath。同样，cpw也发布了一个[视频](https://www.youtube.com/watch?v=pLWQk6ed56Q)来解释这些步骤。

1. 在工程视图中选择`src/main/test`目录，并从菜单中运行`Build -> Build module 'Forge_test'`
2. 打开`File -> Project Structure`内的“Project Structure”窗口
3. 前往“Modules”选项，展开`Forge`模组
4. 选择`Forge_test`子模组，前往“Paths”面板
5. 记住"Test output path"标签内的路径，从树中选择`Forge_main`子模组
6. 打开“Dependencies”面板，点击右侧的绿色加号按钮，并选择“JARs or directories”
7. 选择之前显示在`Forge_test`输出路径中的路径，点击确认
8. 将新添加依赖项的“Scope”设置为“Runtime”（当前为“Compile”），因为主代码编译并不依赖于测试代码

现在你已经添加测试Mod到类路径中了，你需要在每次做出变动时重新构建它们，它们是不会自动构建的。要想构建的话，重复步骤1，或者，如果你对测试Mod文件进行了修改，想要重新构建，只需要点击`Build -> Rebuild project`或者对应的键盘快捷键（默认是Ctrl+F9）就可以了。

#### 使用已有的Mod测试

你可能会想在已有的工程中测试Forge的变动。在测试Mod部分中提到的cpw的视频也讨论了这一部分。让Mod运行需要和测试Mod类似的步骤，但添加你的工程到工作空间还需要一些附加的工作。

1. 打开`File -> Project Structure`内的“Project Structure”窗口
2. 前往“Modules”部分，按下树视图上方的绿色加号图标
3. 选择“Import Module”，选择你的工程的`build.gradle`文件，并确认选择以及导入设置
4. 关闭“Project Structure”窗口，点击“OK”按钮
5. 在IDEA导入完成之后，重新打开窗口，并从树中选择你工程的`_main`模组
6. 打开“Dependencies”面板，点击右侧的绿色加号按钮，并选择“Module dependency”
7. 在刚打开的窗口中，选择`Forge_main`模组
8. 在这之后，重复测试Mod部分中的步骤，只是记得将`Forge_test`替换为你的工程的`_main`模组

!!! note

	你可能会想要移除正常开发环境中的已有的依赖项（主要是指的`forgeSrc`这个Jar），或者将Forge模组移动到依赖项列表的上方。

你现在应该就能够使用对Forge和原版代码的改动，对你的Mod进行测试了。

作出改动和Pull Request
---------------------

一旦配置好了开发环境，就是时候对Forge的代码库进行一些改动了。然而，在修改工程的代码时你应该避免这些事项。

最重要的一点就是，如果你希望修改Minecraft的源代码，你必须在“Forge”这个子工程中进行。任何在“Clean”工程中的改动都将损坏ForgeGradle和生成补丁的过程。这可能会带来灾难性的后果，并且让你的开发环境变得完全无用。如果你希望有一个完美的体验，请保证仅仅修改“Forge”工程中的代码。

### 生成补丁

在你对代码作出变动并仔细测试之后，你就可以生成补丁(Patch)了。这仅仅在你对Minecraft代码（即在“Forge”工程中）进行开发时才需要，但这一步是让你的变动在其它机器上能够正常运行的重要步骤。Forge是通过对原版Minecraft注入改动的代码来工作的，所以它需要将改动储存为一个合适的格式。幸运的是，ForgeGradle能够为你生成改动集，你所需要做的只是commit它们。

如果需要开始生成补丁，请在IDE中或者命令行中运行`genPatches`这个Gradle任务。在它完成之后，你就可以commit所有的变动（保证你没有添加任何不需要的文件）并提交你的Pull Request了。

### Pull Request

提交变动到Forge前的最后一步就是Pull Request了（简写PR）。这是将你Fork的代码融入到正式代码库的一个正式请求。创建PR是很容易的，只需要前往[这个GitHub页面](https://github.com/MinecraftForge/MinecraftForge/compare)，并按照指定的步骤操作即可。在这里分支的重要性就体现出来了，你能够准确地选择你需要提交的改动。

!!! note

	Pull Request是遵循一些规则的，不是所有的请求都会被盲目地接受。请参考[这篇文档](https://github.com/MinecraftForge/MinecraftForge/blob/1.10.x/CONTRIBUTING.md)来获得更多的信息，并保证你的PR的质量！如果你希望将PR被接受的几率最大化，请遵循这个[PR指南](prguidelines.md)！