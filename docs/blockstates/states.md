方块状态介绍
===========

请在开始写代码前将本指南**全部**阅读完。你的理解将会比你阅读部分只更加全面和正确。

这个指南旨在对方块状态(Block States)提供一个入门等级的介绍。如果你知道什么是拓展状态(Extended States)，你可能会注意到我精简了一下东西。这是有意的，我们不想给初学者太多不会及时用到的信息。如果你不知道什么是拓展状态，不要担心，我们在之后会另外写一个文档。

使用方块状态的原因
----------------

在Minecraft 1.8中，方块和Metadata值的直接操作被抽象成了方块状态。  
这个系统的目的为避免难看又不直观的原始Metadata的操作。

举个例子，看下面这个switch语句，它是任意一个有朝向的半砖方块：

```java
switch(meta) {
  case 0: // 面向南方，在方块的下半部分
  case 1: // 面向南方，在方块的上半部分
  case 2: // 面向北方，在方块的下半部分
  case 3: // 面向北方，在方块的上半部分
  ... etc.
}
```

这些数字本身没有任何意义。如果没有注释，我们根本就没法知道那些数字代表了什么。

新的思考方式
-----------

与其到处用一些意义不明的数字，我们不如使用一些系统来使保存的数据从方块语义本身抽象出来。这就是为什么我们现在会有 `IProperty<?>`。每一个 `Block` 都有一组0个或更多个这样的对象。自然它描述了方块的属性(Properties)。比如说，方块可能包含颜色（`IProperty<EnumDyeColor>`），朝向（`IProperty<EnumFacing>`），整数或boolean值等。每一个属性都可以有一个在 `IProperty` 内指定的类型的**值**。比如说，对于我们之前例子里的属性，我们可以有值 `EnumDyeColor.WHITE`，`EnumFacing.EAST`，`1`，或 `false`。

接下来，我们可以看到每一个特定的三元组（Block，属性集合，属性值的集合）都是一个对Block和Metadata的合适并且抽象的替代物。现在，我们有了“minecraft:stone_button[facing=east,powered=true]”，而不是 “minecraft:stone_button meta 9”，猜猜两个之中哪个更明确？

我们给了这个三元组一个特殊的名字——它们叫做 `IBlockState`。

为你的方块添加这些神奇的属性
------------------------

既然我已经成功地说服你属性和值比数字要更好了，我们接下来将会介绍如何使用这一系统。

在你的 `Block` 类，为方块的每个属性都创建一个 `static final` 的 `IProperty<>` 对象。原版提供给我们几个方便的实现：

- `PropertyInteger`：实现 `IProperty<Integer>`。通过调用 `PropertyInteger.create("<name>", <min>, <max>);` 来创建。
- `PropertyBool`：实现 `IProperty<Boolean>`。通过调用`PropertyBool.create("<name>");` 来创建。
- `PropertyEnum<E extends Enum<E>>`：实现 `IProperty<E>`，定义了一个包含Enum类值的属性。通过调用 `PropertyEnum.create("name", <enum_class>);` 来创建。
  - 你也可以只用Enum值的一个子集（比如说你可以只用16个 `EnumDyeColor` 值中的4个值。你可以看一看 `PropertyEnum.create` 的其它重载）。
- `PropertyDirection`：这是 `PropertyEnum<EnumFacing>` 的一个简便实现。
  - 也有提供一些简便的限制(Predicate)。比如说，如果要获取一个表示主方向的属性，你可以调用 `PropertyDirection.create("<name>", EnumFacing.Plane.HORIZONTAL)`。如果你需要获取X方向，调用 `PropertyDirection.create("<name>", EnumFacing.Axis.X)`

注意你也可以制作你自己的 `IProperty<>`，但具体的方法将不会在这篇文章里讲到。

另外，注意你可以共享同一个 `IProperty` 对象到不同的方块中。原版通常会分开放在每一个方块内，但这仅仅是个人喜好。

!!! note "提示"

	如果你的mod有一个API，或者需要与其他mod进行互动。我**非常非常**建议你将你的 `IProperty`(及其值的类) 储存在你的API中。这样的话用户就可以使用方块状态来改变你的方块，而不是用一些意义不明的Metadata。

在你创建了 `IProperty<>` 对象之后，你还需要在 `Block` 类中重写 `createBlockState`。这个方法中，你只需要写一句 `return new BlockState()`。`BlockState` 构造器第一个参数是你的方块，`this`，之后的参数是你想要声明的所有 `IProperty`。注意在1.9及以后的版本中，`BlockState` 类被改名为 `BlockStateContainer`，这其实更准确地描述了这个类的实际作用。

你刚刚创建的这个对象非常神奇——它管理了所有三元组的生成。也就是说，它会对每一个属性每一个值生成所有可能的组合（如果你数学比较好，它获取了每一属性所有可能值的集合，并计算所有属性值集合的笛卡尔积）。所以，它将会生成所有可能的不同（Block，属性，值）三元组——一个属性所有可能的 `IBlockState`。

如果你不设置其中一个 `IBlockState` 来作为方块的默认状态，那么则会随机选中一个。你可能不会想要这个情况发生（这通常会造成奇怪的情况），所以在你的Block构造器最后，调用 `setDefaultState()`，并传递你想要作为默认值的 `IBlockState`。你可以使用 `this.blockState.getBaseState()` 获得被选中的那个，并对**每一个**属性都用 `withProperty` 设置值。

由于 `IBlockState` 是不可变和预先生成的，调用 `IBlockState.withProperty(<PROPERTY>, <NEW_VALUE>)` 将会转到 `BlockState`/`BlockStateContainer` 并请求(Request)一个对应你需求值的 `IBlockState`，而不是创建一个新的 `IBlockState`。

由于基本的 `IBlockState` 在启动时就生成为固定的集合，所以通过引用相等(==)来比较 `IBlockState` 是完全安全并且推荐的。

使用 `IBlockState`
-----------------

我们知道，`IBlockState` 是个非常强大的对象，你可以通过调用 `getValue(<PROPERTY>)` 来获取一个属性的值，传递的参数为你想要检测的 `IProperty<>`。

如果你想获取一个拥有不同集合值的 `IBlockState`，调用 `withProperty(<PROPERTY>, <NEW_VALUE>)`。这将会返回另一个预先生成的对应 `IBlockState` 对象。

你可以使用 `setBlockState()` 和 `getBlockState()` 在世界中获取或设定 `IBlockState`。

幻想杀手
--------

很遗憾，抽象层在其核心其实是个谎言。我们仍需要翻译每一个 `IBlockState` 到一个从0到15的数字，它将会储存在世界中或者从世界中读取。

如果你声明了任何 `IProperty`，你**必须**重写 `getMetaFromState` 和 `getStateFromMeta`。

这里你将会读取属性的值，并返回一个合适的0到15之间的整数，或者是倒过来。读者们可以自己看看原版方块的例子。

!!! warning "警告"

	你的 `getMetaFromState` 和 `getStateFromMeta` 方法**必须**是一对一的！换句话说，相同的属性与值必须映射到同一meta值，反之亦然。如果你没有做到这个，游戏将**不会**崩溃。它只会让所有东西都表现得很奇怪。

“实际”状态
---------

你可能会注意到栅栏并不将连接属性记录到meta中，但在F3菜单中你仍可以看到它的属性和值！这是为什么呢？

方块可以声明不保存到Metadata中的属性。这些通常用作渲染目的，但也可以有一些其他的用处。

你仍可以在 `createBlockState` 中声明它们，并且使用 `setDefaultState` 设置值。然而，这些值在 `getMetaFromState` 和 `getStateFromMeta` 里都**不要**动。

然而，你需要在你的 `Block` 类中重写 `getActualState`。这里你将会接收到对应世界中Metadata值的 `IBlockState` ，你需要返回另一个包含缺失信息的 `IBlockState`，比如说栅栏链接，红石链接等。你需要用 `withProperty` 来附加属性。你也可以用这个来对一个值读TileEntity数据（当然你需要恰当的安全检查！）。

!!! warning "警告"

	当你在 `getActualState` 中读取TileEntity的数据时，你必须采取进一步的安全检查。默认情况下，`getTileEntity` 将会在TileEntity不存在时尝试创建这个TileEntity。然而，`getActualState` 和 `getExtendedState` 可以并且会被另外的线程调用，这将会导致在它尝试创建丢失TIleEntity时，世界的TileEntity列表会抛出 `ConcurrentModificationException`。所以，你必须检查 `IBlockAccess` 参数是否为 `ChunkCache`（传给其它线程的对象），如果是的话，将其强制转换为 `ChunkCache` 类型，并使用 `getTileEntity` 不可写的变种。安全检查的例子可以在 `BlockFlowerPot.getActualState()` 中找到。

!!! note "提示"

	调用 `world.getBlockState()` 将会返回代表metadata的 `IBlockState`。所以返回的 `IBlockState` 将不会包含 `getActualState` 内的数据。如果这写数据对你的代码很重要，请务必调用 `getActualState`！

拓展阅读
-------

- 1.8+中的渲染：未完成
- 拓展状态(Extended States)：未完成
