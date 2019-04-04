模型文件
===========

一个“模型”可以是一个简单的形状，可以是几个长方体，可以是大斜方截半二十面体，或者其它形状。你见到的绝大多数模型是以原版的JSON的格式的。其它格式的模型是运行时由`ICustomModelLoader`加载到`IModel`中。Forge默认提供WaveFront OBJ和Blitz3D的实现。大多数时候不用担心模型是什么格式的，因为它们在代码中都实现了`IModel`接口。

当`ResourceLocation`指的是一个模型时，路径会关联到`models`下(例如 :`examplemod:block/block` → `assets/examplemod/models/block/block`)。常见错误是在[方块状态JSON][blockstate JSON](共3种格式)中模型文件会关联到`models/block` (例如: `examplemod:block` → `assets/examplemod/models/block/block`)。

方块和物品的模型有一点不同，最主要在[物品属性概述][overrides]。

材质
--------

材质，和模型一样，包括在材质包中，用`ResourceLocation`表示。当`ResourceLocation`在模型中表示的是材质，路径会关联到`textures/`(例如: `examplemod:blocks/test` → `assets/examplemod/textures/blocks/test.png`)。另外，在Minecraft中，[UV坐标][UV]中，(0,0)代表__左上__角。UV_总是_在0到16之间。如果材质更大或更小，坐标要适当缩放，材质必须是正方形的，且边长最好是2的幂，因为不这样的话会破坏纹理映射。(例如: 1x1, 2x2, 8x8, 16x16, 和128x128是最好的, 5x5和30x30不推荐因为边长不是2的幂，5x10和4x8会完全损坏因为不是正方形)。如果存在与纹理相关联的`mcmeta`文件，并且定义了动画，则图像可以是矩形的，会被解释为从上到下的正方形区域的垂直序列，其中每个正方形是动画的帧。

JSON 模型
-----------

Minecraft原版的JSON模型格式非常简单。它定义了长方体(矩形棱柱)元素，并为其面指定材质。在[wiki][JSON model format]中有它格式的定义

!!! note "提示"

    JSON模型只支持立方体元素；没法表示三棱柱或其它这样的东西。要有更复杂的东西，必须用其它格式。

当一个`ResourceLocation`只JSON模型的位置，它没有后缀`.json`，不像OBJ和B3D模型(例如:  `minecraft:block/cube_all`, 不是`minecraft:block/cube_all.json`)

WaveFront OBJ 模型
--------------------

Forge增加了`.obj`格式文件的加载器。要用这些模型，资源的命名空间必须用 `OBJLoader.addDomain`注册。加载器接受注册过的命名空间下的以`.obj`结尾的任何文件。`.mtl`文件应放在`.obj`文件旁，并会在使用`.obj`是自动使用。可能必须要手动编辑`.mtl`文件把其纹理指向Minecraft的`ResourceLocation`。另外，用外部软件创建的材质的V轴可能被翻转了(例如，V=0可能是底部，而不是顶部)。 这可以在建模程序本身中纠正，或者在Forge方块状态JSON中完成，如下所示：

```json
{
  "__comment": "在与“模型”声明相同的级别上添加以下行。",
  "custom": { "flip-v": true },
  "model": "examplemod:model.obj"
}
```

Blitz3D 模型
--------------

Forge增加了`.b3d`格式文件的加载器。要用这些模型，资源的命名空间必须用 `B3DLoader.addDomain`注册。加载器接受注册过的命名空间下的以`.b3d`结尾的任何文件。

[JSON model format]: https://minecraft.gamepedia.com/Model#Block_models
[overrides]: overrides.md
[blockstate JSON]: blockstates/introduction.md
[UV]: https://en.wikipedia.org/wiki/UV_mapping
