构造您的mod
====================

我们将在这一节看看如何管理你的mod到不同的文件，并且这些文件能干什么。

包(Packaging)
---------

选择一个独一无二的包的名字。如果你拥有一个与你的工程关联的URL，你可以把它当做你顶级包(Top-level package)。比如说你拥有一个网站 "fakemods.com" ，你可以将 `com.fakemods` 当做你的顶级包。

!!! important

    如果你没有一个域名，不要用它当做你的顶级包的名字。用任何东西命名你的包都是可以接受的，比如说你的名字/昵称，或者是mod的名字。

在顶级包之后(如果你有一个)你需要添加一个你唯一的mod名字，比如说 `examplemod`。在这里这个包将会最终是 `com.fakemods.examplemod`。

Mod文件
-------

通常情况下，我们将会使用一个以你的mod名字来命名的文件开始，将其放入你的包下面。这将是你的mod的**入口点(Entry Point)**，并且将会包含一些特殊的指示来标记它。

什么是 `@Mod`？
-------------

这是一个注解(Annotation)，告诉Forge Mod Loader(FML)这个类(Class)是一个Mod的入口点。它可以包含不同的关于这个mod的元数据(Metadata)。它也同样指示了这个类将会收到 `@EventHandler` 事件。更多的信息在这里...(未完成)

你可以在 [Forge src download](http://files.minecraftforge.net/)找到一个示例mod。

使用子包保持代码整洁
------------------

我们推荐您分解你的mod到不同的子包而不是堆在单独一个类和包里面。

我们通常把包分成 `common` 和 `client` 两部分，分别对应运行在服务器/客户端(`common`)与客户端(`client`)的代码。在 `common` 包里面应该放物品、方块、和Tile Entity(可以每个都再分到一个子包里面)。而GUI和渲染相关代码应该放在 `client` 包里面。

!!! note

    这种分包风格只是一种建议，尽管这是非常常用的一种风格。你也可以自由地选择你自己的分包风格。

如果你将代码干净地分在子包中，你就能更快地发展你的mod。

命名风格
-------

一个通用的命名风格能让你更容易知道一个类是干什么用的，并且能方便你的合作者来寻找东西。

例如:

- 一个叫做 `PowerRing` 的 `Item`(物品)应该放在 `item` 包下，并且类名为 `ItemPowerRing`
- 一个叫做 `NotDirt` 的 `Block`(方块)应该放在 `block` 包下，类名为 `BlockNotDirt`
- 最后，一个叫做 `SuperChewer` 的方块的 `TileEntity` 应该放在 `tile` 或者是 `tileentity`包下，类名为 `TileSuperChewer`

在你的类名前面加上这个对象所属的**种类**让人更容易弄清楚这个类是什么，或者通过一个对象来猜测类的名字。