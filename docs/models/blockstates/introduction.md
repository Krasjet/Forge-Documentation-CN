方块状态JSON概述
================================

方块状态JSON是Minecraft将“变体字符串”映射到模型的方式。 变体字符串可以是任何东西，如“inventory”,“power= 5”或“I am your father”。 它们代表一个实际模型，其中方块状态只是它们的容器。 在代码中，方块状态JSON中的变体字符串由`ModelResourceLocation`表示。

当游戏搜索对应于世界中某个方块的模型时，它会对该位置采取[方块状态][blockstate]，然后使用`IStateMapper`为它找到相应的`ModelResourceLocation`，然后引用它 实际模型。 默认的`IStateMapper`使用方块的注册表名称作为方块状态JSON的位置。 （例如，方块`examplemod:testblock`对应的`ResourceLocation`是`examplemod:testblock`）变体字符串把方块状态的属性拼凑在一起。 更多信息可以在[这里][statemapper]找到。

举个例子,我们来看看原版的`oak_log.json`:

```json
{
  "variants": {
    "axis=y":  { "model": "oak_log" },
    "axis=z":   { "model": "oak_log_side" },
    "axis=x":   { "model": "oak_log_side", "y": 90 },
    "axis=none":   { "model": "oak_bark" }
  }
}
```

在这里我们定义了4个变体字符串，每个我们使用一个特定的模型，直立原木，侧立原木（旋转或不旋转）和全树皮模型（这种模型通常不会在原版中看到;你必须使用` /setblock`来创建它）。由于原木使用默认的`IStateMapper`，因此这些变体将根据属性`axis`定义原木的外观。

必须始终为所有可能的变体字符串定义块状态。 如果有许多属性，则会产生许多可能的变体，因为必须定义每个属性组合。 在Minecraft 1.8的方块状态格式中，您必须明确定义每个字符串，这会导致长而复杂的文件。 它也不支持子模型的概念，也不支持同一个方块状态中的多个模型。 为了实现这一目标，Forge推出了[自己的blockstate格式][Forge blockstate]，可以在Minecraft 1.8及更高版本中使用。

从Minecraft 1.9开始，Mojang还推出了“multipart”格式。 您可以在[wiki][]上找到其格式的定义。 Forge的格式和multipart格式并没有谁比谁更好; 它们各自涵盖不同的用例，您可以选择使用哪种用例。

!!! note "提示"

    Forge格式更像是语法糖，用于在后台自动计算所有可能变体的集合。 这允许您使用生成的`ModelResourceLocation`来处理方块之外的其他内容。（例如[item][item blockstates]。1.8格式也是如此，但几乎没有理由使用那种格式。）1.9格式是一个更复杂的系统，依赖于`IBlockState` 选择模型。 如果没有代码，它将不会直接在其他环境中工作。

作为参考，这里是1.8栅栏的方块状态`fence.json`的摘录：

```json
"east=true,north=false,south=false,west=false": { "model": "oak_fence_n", "y": 90, "uvlock": true }
```

这只是16个中的一个变体。更糟糕的是，有6个围栏模型，一个用于无连接，一个连接，两个直线连接，两个垂直连接，三个连接，以及一个用于所有四个连接。

这是1.9中同样的例子,但用的是multipart格式:

```json
{ "when": { "east": "true" },
  "apply": { "model": "oak_fence_side", "y": 90, "uvlock": true }
}
```


这是一个5的情况。您可以将其读作“当east = true时，使用模型oak_fence_side旋转90度”。 这允许最终模型由5个较小的部分构成，其中4个（连接）是有条件的，第5个是无条件的中心柱。 这仅使用两个模型，一个用于中心柱，一个用于侧面连接。

[blockstate]: ../../blocks/states.md
[statemapper]: ../using.md#block-models
[Forge blockstate]: forgeBlockstates.md
[wiki]: https://minecraft.gamepedia.com/Model#Block_states
[item blockstates]: ../using.md#blockstate-jsons-for-items
