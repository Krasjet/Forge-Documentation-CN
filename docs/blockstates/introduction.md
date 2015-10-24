方块状态(Blockstates)简介
===========================

方块状态(Blockstates)告诉游戏根据方块的数据应当使用哪一个模型。
一个简单的方块只有**标准(normal)**一个默认的方块状态。
一个有多种显示方式的更加复杂的方块(译注: 比如能够旋转的方块)有**变种(variants)**。

举个例子，我们来看一看原版的 `oak-log.json`:

```json
{
    "variants": {
        "axis=y":    { "model": "oak_log" },
        "axis=z":     { "model": "oak_log_side" },
        "axis=x":     { "model": "oak_log_side", "y": 90 },
        "axis=none":   { "model": "oak_bark" }
    }
}
```

你可以看到它并没有标准状态，只有依赖于"axis"值的不同的变种。根据木头被对齐的轴，它会使用正立的模型，横向的模型(旋转90°或者没有旋转的)，或者是，如果这个方块的轴没有被设定，六面全是树皮的模型。

这个例子中木头只有一个属性: `axis`。定义一个状态必须使用一个方块的所有的属性。这会很快导致变种组合的爆炸。我们来看一看原版中栅栏的其中一个变种:

```json
"east=false,north=false,south=false,west=false": { "model": "oak_fence_post" }
```

而这仅仅是其16个变种中的一个。这会很快导致非常大并且冗长的blockstate文件，这也是Minecraft 1.8的主要问题。Minecraft 1.9将会引入一个系统使这个问题得到控制。[Forge's Blockstate Json][forge]让你能在1.8里面解决这个问题。

[forge]: forgeBlockstates.md "Forge's Blockstate Json"
