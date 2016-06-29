方块(Block)
==========

很显然，方块(Block)在Minecraft世界中非常重要。它们组成了所有的地形，结构，以及机械装置。如果你对制作mod很感兴趣，那么你很可能会想加一些方块。这一节将指导你创建方块，并让你了解一些方块的功能。

创建一个方块
-----------

### 简单方块

对于简单的方块，或者说那些没有特殊功能的方块(如圆石和木板等)，你并不需要对它们单独创建一个类。你只需要实例化一个 `Block` 类并调用一些setter方法，就能创建出很多不同种类的方块。比如说：

- `setHardness` - 控制破坏方块所需要的时间。它可以是任意的值。方便参考，石头(Stone)的硬度(Hardness)为1.5，泥土(Dirt)的硬度为0.5。如果方块应该是不可破坏的，你可以直接调用 `setBlockUnbreakable`。
- `setResistance` - 控制方块的爆炸抗性。虽然这个值是与硬度分开的，但是 `setHardness` 同样会将方块的爆炸抗性(Resistance)设置为5倍的硬度，如果爆炸抗性低于这个值的话。(译注：这个方法设置后的实际 `blockResistance` 是传入参数的3倍。 比如说石头的爆炸抗性(`blockResistance`)为30，但传入的参数为 `10.0f`。而`setHardness` 检测的是这个 `blockResistance`。但一般初始化方块的时候一般都先设置硬度再设置爆炸抗性)
- `setSoundType` - 控制方块被击打，破坏，或放置时的声音。需要传入一个 `SoundType` 参数，具体请看[音效]那一节。
- `setLightLevel` - 控制方块的发光程度。**注意：**这个方法接受一个0-1的值，不是0-15。如果想要计算这个值，请用你需要的光亮等级除以16。比如说，如果一个方块的亮度为5，那么这里的参数需要为 `5 / 16f`。
- `setLightOpacity` - 控制方块阻碍光线传播的程度。不像 `setLightLevel` 这个值的范围是0-15。比如说，将这个值设定为 `3` 时，每当光线穿过方块，光线的光亮等级将会减少3级。
- `setUnlocalizedName` - 设置方块未本地化时(Unlocalized)的名字。这个名字会被前缀 "tile."，并后缀 ".name" 以方便本地化。比如说，`setUnlocalizedName("foo")` 会将方块的本地化条目设置为 "tile.foo.name"。如果你需要更高级的本地化控制，那么你还需要一个自定义的物品(Item)类，我们将会在之后进行介绍。
- `setCreativeTab` - 控制方块所在的创造面板。面板选项可以在 `CreativeTabs` 类下找到。

所有的这些方法都是可以**可链接**的，也就是说你可以以一个序列调用这些方法。如果想要一个例子，请看 `Block#registerBlocks` 方法。

### 高级方块

当然，上面介绍的东西仅仅只能创建一个非常简单的方块。如果你想添加更多的功能，比如说玩家互动，你需要一个自定义的类。然而，`Block` 类有太多方法了，很遗憾这里不能将每一个都列在这里并加以解释。请阅读这节的剩余部分来看看你可以对方块添加什么功能。

注册方块
-------

好了，现在你已经创建好一个方块类，现在该将其放到游戏里了。这是个很简单的事情。

和大多数东西一样，方块可以使用 `GameRegistry.register(...)` 来注册。这个方法中，你的方块需要有一个“注册名(Registry Name)”，或者说是方块的唯一名字。推荐在 `register` 方法内调用 `setRegistryName`来设置注册名。比如说 `GameRegistry.register(myBlock.setRegistryName("foo"))`。

!!! note

	当使用一个简单String的时候，当前的mod的ID将会被添加作为前缀。所以，如果我是在 "mymod" 中注册的，真实的注册名会是 "mymod:foo"。

直接将一个 `ResourceLocation` 传入 `register` 也是可以的，但调用 `setRegistryName` 并传入一个事先准备好的 `ResourceLocation` 会更方便一点。

拓展阅读
-------

对于更多方块属性的信息，比如说原版的木头类型，栅栏，墙等，请阅读[方块状态]小节。

[音效]: ../effects/sounds.md
[方块状态]: ../blockstates/states.md