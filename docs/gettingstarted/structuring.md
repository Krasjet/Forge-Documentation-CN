构造您的mod
==========

我们将在这一节看看如何管理你的mod到不同的文件，并且这些文件能干什么。

包(Packaging)
-------------

选择一个独一无二的包的名字。如果你拥有一个与你的工程关联的URL，你可以把它当做你顶级包(Top-level package)。比如说你拥有一个网站 "example.com" ，你可以将 `com.example` 当做你的顶级包。

!!! important

    如果你没有一个域名，不要用它当做你的顶级包的名字。用任何东西命名你的包都是可以接受的，比如说你的名字/昵称，或者是mod的名字。

在顶级包之后（如果你有一个）你需要添加一个你唯一的mod名字，比如说 `examplemod`。在这里这个包将会最终是 `com.example.examplemod`。

`mcmod.info`文件
----------------

这个文件定义了你的mod的元数据(Metadata)。玩家可以从游戏主菜单的Mods按钮内看到这些信息。一个info文件可以描述多个Mod的信息。当一个Mod使用 `@Mod` 注解的时候，它可以定义一个 `useMetadata` 属性，默认是为 `false` 的。当 `useMetadata` 为 `true` 的时候，`mcmod.info` 文件中的元数据将会覆盖注解中所定义的信息。

`mcmod.info` 文件为JSON格式，它的根元素为一系列的对象，每个对象描述一个modid。它应该储存在 `src/main/resources/mcmod.info`。一个基本的描述单mod的 `mcmod.info` 可能会像这样子：

```json
[{
  "modid": "examplemod",
  "name": "Example Mod",
  "description": "Lets you craft dirt into diamonds. This is a traditional mod that has existed for eons. It is ancient. The holy Notch created it. Jeb rainbowfied it. Dinnerbone made it upside down. Etc.",
  "version": "1.0.0.0",
  "mcversion": "1.10.2",
  "logoFile": "assets/examplemod/textures/logo.png",
  "url": "minecraftforge.net/",
  "updateJSON": "minecraftforge.net/versions.json",
  "authorList": ["Author"],
  "credits": "I'd like to thank my mother and father.",
}]
```

默认的Gradle配置将会把 `${version}` 替换为工程的版本，并把 `${mcversion}` 替换为Minecraft版本，但替换**仅**会发生在 `mcmod.info` 文件中，所以你应该使用这些替代字符串而不是明文写出来。下面这个表格列出了所有可能的属性，其中 `必填` 表示没有默认值可用，如果留空的话则会造成错误。除了必填的属性，你至少还应该定义 `description`，`version`，`mcversion`，`url` 和 `authorList`。

|     属性 |   类型   | 默认值  | 简介 |
|-------------:|:--------:|:--------:|:------------|
|        modid |  string  | 必填 | 这篇介绍所链接的modid。如果这个Mod没有加载，那么这篇介绍将会被忽略 |
|         name |  string  | 必填 | Mod显示的名字 |
|  description |  string  |   `""`   | 1-2段话的Mod简介 |
|      version |  string  |   `""`   | Mod的版本 |
|    mcversion |  string  |   `""`   | Minecraft的版本 |
|          url |  string  |   `""`   | Mod主页的链接 |
|    updateUrl |  string  |   `""`   | 有定义但没有实际用处。现在被updateJSON取代了 |
|   updateJSON |  string  |   `""`   | [version JSON](autoupdate#forge-update-checker)的地址 |
|   authorList | [string] |   `[]`   | Mod作者的列表 |
|      credits |  string  |   `""`   | 可以包含任何你想感谢的东西 |
|     logoFile |  string  |   `""`   | Mod Logo的地址。它会根据classpath来解析，所以你可以把它放在任何不会造成命名冲突的位置，比如说你的assets文件夹中 |
|  screenshots | [string] |   `[]`   | 在信息页面会显示的图片，现在还没有实现 |
|       parent |  string  |   `""`   | 父Mod的modid，如果可用的话。使用这个可以让别的Mod的模块在信息页面列在这个Mod的下面，比如说BuildCraft中就有用到 |
| useDependencyInformation |  boolean |  `false` | 如果设置为true并且 `Mod.useMetadata` 也设置为true，Mod将会使用下面这3个依赖列表。如果不是true的话下面3个值则不会生效 |
| requiredMods | [string] |   `[]`   | 一个modid列表。如果缺少一个，游戏将会弹出提示并停止加载。**这并不会影响到Mod<font color=red>加载</font>顺序！**如果想要同时注明加载顺序和依赖，请在 `dependencies` 中添加相同的条目 |
| dependencies | [string] |   `[]`   | 一个modid列表。所有列出的Mod将会在当前Mod**加载前**加载。如果这一项为空，则什么都不会发生。 |
|   dependants | [string] |   `[]`   | 一个modid列表。所有列出的Mod将会在当前Mod**加载后**加载。如果这一项为空，则什么都不会发生。 |

[BuildCraft](http://gist.github.com/anonymous/05ad9a1e0220bbdc25caed89ef0a22d2) 中有很好的例子，它使用了上述的很多属性。

Mod文件
------

通常情况下，我们将会使用一个以你的mod名字来命名的文件开始，将其放入你的包下面。这将是你的mod的**入口点(Entry Point)**，并且将会包含一些特殊的指示来标记它。

什么是 `@Mod`？
-------------

这是一个注解(Annotation)，告诉Forge Mod Loader(FML)这个类(Class)是一个Mod的入口点。它可以包含不同的关于这个mod的元数据(Metadata)。它也同样指示了这个类将会收到 `@EventHandler` 事件。更多的信息在这里...（未完成）

你可以在 [Forge src download](http://files.minecraftforge.net/)找到一个示例mod。

使用子包保持代码整洁
------------------

我们推荐您分解你的mod到不同的子包而不是堆在单独一个类和包里面。

我们通常把包分成 `common` 和 `client` 两部分，分别对应运行在服务器/客户端(`common`)与客户端(`client`)的代码。在 `common` 包里面应该放物品、方块、和Tile Entity（可以每个都再分到一个子包里面）。而GUI和渲染相关代码应该放在 `client` 包里面。

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