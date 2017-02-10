文件位置
=======

Minecraft会要求你将工程的某些文件放在特定的位置上，比如说材质文件和JSON。

这节教程中所有的位置以及文件都是对于 `./src/main/resources/` 文件夹的**相对位置**.

### mcmod.info

`mcmod.info` 文件放在根目录。

### 方块状态

方块状态(Blockstates)定义文件（JSON格式）放在 `./assets/<modid>/blockstates/` 文件夹中。

### 本地化文件

本地化文件(Localization)是后缀名为 `.lang` 格式的纯文本文件，它们的文件名为语言各自的[语种代码](https://msdn.microsoft.com/en-us/library/ee825488(v=cs.20).aspx)，比如说 `en_US`。

它们放在 `./assets/<modid>/lang/` 文件夹中。

### 模型

模型(Models)文件为JSON格式，分别放在 `./assets/<modid>/models/block/`（方块）或 `./assets/<modid>/models/item/`（物品）文件夹内。

### 材质

材质(Textures)文件为PNG格式，分别放在 `./assets/<modid>/textures/blocks/`（方块）或 `./assets/<modid>/textures/items/`（物品）文件夹内。