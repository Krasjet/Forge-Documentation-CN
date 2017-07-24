方块
==========

很显然，方块(Block)在Minecraft世界中非常重要。它们组成了所有的地形，结构，以及机械装置。如果你对制作mod很感兴趣，那么你很可能会想加一些方块。这一节将指导你创建方块，并让你了解一些方块的功能。

创建一个方块
-----------

### 简单方块

对于简单的方块，或者说那些没有特殊功能的方块（如圆石和木板等），你并不需要对它们单独创建一个类。你只需要实例化一个 `Block` 类并调用一些Setter方法，就能创建出很多不同种类的方块。比如说：

- `setHardness` - 控制破坏方块所需要的时间。它可以是任意的值。方便参考起见，石头(Stone)的硬度(Hardness)为1.5，泥土(Dirt)的硬度为0.5。如果方块应该是不可破坏的，你可以直接调用 `setBlockUnbreakable`。
- `setResistance` - 控制方块的爆炸抗性。虽然这个值是与硬度分开的，但是 `setHardness` 同样会将方块的爆炸抗性(Resistance)设置为5倍的硬度，如果爆炸抗性低于这个值的话。（译注：这个方法设置后的实际 `blockResistance` 是传入参数的3倍。 比如说石头的爆炸抗性(`blockResistance`)为30，但传入的参数为 `10.0f`。而`setHardness` 检测的是这个 `blockResistance`。但一般初始化方块的时候一般都先设置硬度再设置爆炸抗性）
- `setSoundType` - 控制方块被击打、破坏、或放置时的声音。需要传入一个 `SoundType` 参数，具体请看[音效]那一节。
- `setLightLevel` - 控制方块的发光程度。**注意：**这个方法接受一个0-1的值，而不是0-15。如果想要计算这个值，请用你需要的光亮等级除以16。比如说，如果一个方块的亮度为5，那么这里的参数需要为 `5 / 16f`。
- `setLightOpacity` - 控制方块阻碍光线传播的程度。不像 `setLightLevel`，这个值的范围是0-15。比如说，将这个值设定为 `3` 时，每当光线穿过方块，光线的光亮等级将会减少3级。
- `setUnlocalizedName` - 设置方块未本地化时(Unlocalized)的名字。这个名字会被前缀 "tile."，并后缀 ".name" 以方便本地化。比如说，`setUnlocalizedName("foo")` 会将方块的本地化条目设置为 "tile.foo.name"。如果你需要更高级的本地化控制，那么你还需要一个自定义的物品(Item)类，我们将会在之后进行介绍。
- `setCreativeTab` - 控制方块所在的创造面板。面板选项可以在 `CreativeTabs` 类下找到。

所有的这些方法都是可以**可链的**(Chainable)，也就是说你可以以一个序列调用这些方法。如果想要一个例子，请看 `Block#registerBlocks` 方法。

### 高级方块

当然，上面介绍的东西仅仅只能创建一个非常简单的方块。如果你想添加更多的功能，比如说玩家互动，你需要一个自定义的类。然而，`Block` 类有太多方法了，很遗憾这里不能将每一个都列在这里并加以解释。请阅读这节的剩余部分来看看你可以对方块添加什么功能。

注册方块
-------

方块必须要[注册]至函数中。

!!! important

	世界中的方块和物品栏中的“方块”截然不同。世界中的方块由 `IBlockState` 来表示，它的行为由 `Block` 的实例来定义。而物品栏中的物品是一个 `ItemStack`，由 `Item` 来控制。作为 `Block` 与 `Item` 之间的桥梁，`ItemStack` 类诞生了。`ItemStack` 是 `Item` 的一个子类，但它封装了其对应 `Block` 的引用。`ItemBlock` 定义了一个“方块”作为物品的行为，比如说右键如何能放置这个方块。一个 `Block` 没有 `ItemBlock` 也是可以的。（比如说 `minecraft:water` 存在有这个方块，但没有这个物品。所以不能将它放在物品栏中。）

	当你注册一个方块的时候，你**仅**注册了这个方块。这个方块不会自动有一个 `ItemBlock`。为了给你的方块添加一个最基础的 `ItemBlock`，你应该使用 `new ItemBlock(block).setRegistryName(block.getRegistryName())`。它的非本地化名称和方块的是一样的。你也可以使用自定义的 `ItemBlock` 子类。 当 `ItemBlock` 注册到一个方块上之后，可以使用 `Item.getItemFromBlock` 来获取它。如果这个 `Block` 没有 `ItemBlock`，`Item.getItemFromBlock` 将会返回 `null`，所以如果你不确定你要使用的方块是否有 `ItemBlock`，先检查 `null` 值。

对方块着色
---------

方块的纹理是可以在程序中着色的。很多原版的方块都用到了这一功能。比如说：草丛、藤蔓、睡莲等都会根据所处生态群系(Biome)改变它们的颜色。

### 方块颜色处理器

方块颜色处理器是对一个方块着色所必须的东西，它是`IBlockColor`的一个实例，它添加了以下方法：

```java
int colorMultiplier(
  IBlockState state, 
  @Nullable IBlockAccess worldIn, 
  @Nullable BlockPos pos, 
  int tintIndex)
```

#### 返回值

这个方法返回一个十六进制值(Hex)，用整数表示一个颜色。

#### 参数

传入方法的`IBlockState`、`IBlockAccess`和`BlockPos`，让你能够动态改变颜色乘数。注意`IBlockAccess`和`BlockPos`通常是可空的(Nullable)。

### 注册方块颜色处理器

方块颜色处理器必须要通过调用`BlockColors#registerBlockColorHandler(IBlockColor, Block...)`函数来注册。对于多个方块也是可以的。`BlockColors`的实例可以通过调用`Minecraft#getBlockColors()`来获取。

这必须要在初始化阶段中，仅在客户端调用。

### 为`ItemBlock`注册处理器

`ItemBlock`的实例是物品，所以它们必须要像物品一样[注册和着色](../items/items.md#_6)。

拓展阅读
-------

对于更多方块属性的信息，比如说原版的木头类型、栅栏、墙等，请阅读[方块状态]小节。

[音效]: ../effects/sounds.md
[注册]: ../concepts/registries.md#_2
[方块状态]: ../blockstates/states.md