文件位置
=======

Minecraft会要求你将工程的某些文件放在特定的位置上，比如说材质文件和JSON。

这节教程中所有的位置以及文件都是对于 `./src/main/resources/` 文件夹的**相对位置**.

### mcmod.info

`mcmod.info` 文件放在根目录。

### 方块状态(Blockstates)

方块状态定义文件(JSON格式)放在 `./blockstates/` 文件夹中。

### 本地化文件(Localizations)

本地化文件是后缀名为 `.lang` 格式的纯文本文件，它们的文件名为语言各自的[语种代码](https://msdn.microsoft.com/en-us/library/ee825488(v=cs.20).aspx)，比如说 `en_US`。

它们放在 `./lang/` 文件夹中。

### 模型(Models)

模型文件为JSON格式，分别放在 `./models/block/`(方块)或 `./models/item/`(物品)文件夹内。

### 材质(Textures)

材质文件为PNG格式，分别放在 `./textures/blocks/`(方块) 或 `./textures/items/`(物品)文件夹内。