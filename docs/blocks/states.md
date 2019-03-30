方块状态
============

在开始写代码前,请**读完**本指南。读完这篇指南你能理解的更加全面。本指南是介绍方块状态的入门介绍。 如果你知道什么是扩展状态，你会注意到我在下面做了一些简化的假设。 它们是有意的，为了避免初学者一下理解不了全部信息。 如果你不知道它们是什么，不必担心，最后会有另一份文件给他们。

背景
----------

在Minecraft 1.8及更高版本中，对方块和metadata值的直接操作已被抽象成所谓的blockstates。
该系统的前提是删除对没有意义的原始metadata的使用和操作。

例如，考虑这个有方向并占据一半空间的方块的switch语句：

```Java
switch(meta) {
  case 0: // 它朝南,在下部
  case 1: // 它朝南,在上部
  case 2: // 它朝北,在下部
  case 3: // 它朝北,在下部
  ... etc.
}
```

这些数字本身没有任何意义！如果没有注释，就根本不知道这些数字是什么意思。

新的思考方式
---------------------

那么，我们是不是可以不要到处都和数字打交道，而是使用一些系统从方块本身的语义中抽象出保存的细节？这就是`IProperty<?>`的用武之地。每个方块都有一组零个或多个这些对象，可以说这些对象描述了块具有的*属性*。 例如颜色（`IProperty<EnumDyeColor>`），朝向（`IProperty<EnumFacing>`），整数和布尔值等。每个属性都可以表示为* IProperty*类型的*值*。 例如，对于上面的例子，我们可以用值`EnumDyeColor.WHITE`, `EnumFacing.EAST`, `1`, 或 `false`。

那么，这样的话，我们发现方块和metadata可以被唯一的三元组（块，属性的集合，属性的值的集合）替代。 现在，我们不再使用“minecraft:stone_button meta 9”，而是用“minecraft:stone_button[facing=east,powered=true]”。 想想哪个更有意义？

这个三元组有一个特别的名字，叫`IBlockState`。

用这些魔法属性制作你的方块
-------------------------------------------------

既然我已经成功地说服了你键值对优于数字，那么让我们继续讨论实际的操作方法。

In your Block class, create static final `IProperty<>` objects for every property that your Block has. Vanilla provides us several convenience implementations:
在你的Block类中，为Block的每个属性创建静态的final的`IProperty<>`对象。原版客户端为我们提供了几种便利实现:

  * `PropertyInteger`: 实现了 `IProperty<Integer>`.。调用 `PropertyInteger.create("<name>", <min>, <max>)`来创建;
  * `PropertyBool`: 实现了 `IProperty<Boolean>`. 调用 `PropertyBool.create("<name>")`来创建;
  * `PropertyEnum<E extends Enum<E>>`: 实现了 `IProperty<E>`, 定义可以采用Enum类列举的值的属性。 调用`PropertyEnum.create("name", <enum_class>)`来创建;
    * 你也可以只使用Enum的一个子集 (例如，你只要使用16个`EnumDyeColor`中的4个。 可以看看 `PropertyEnum.create`的其它重载)
  * `PropertyDirection`: 这是对 `PropertyEnum<EnumFacing>`的一个方便的实现
    * 还有几个方便的实现. 例如，获取表示基本方向的属性, 你可以调用`PropertyDirection.create("<name>", EnumFacing.Plane.HORIZONTAL)`. 或者获得X方向, `PropertyDirection.create("<name>", EnumFacing.Axis.X)`

In addition, note that you can share the same `IProperty` object between different blocks if you wish. Vanilla generally has separate ones for every single block, but it is merely personal preference.
请注意，您可以自由地创建自己的`IProperty<>`的实现，但本文不涉及实现方法。
另外，请注意，如果您愿意，可以在不同块之间共用相同的“IProperty”对象。 原本通常每个方块都有单独的`IProperty`，但它只是个人喜好。

!!! Note "提示"
    如果你的mod有一个API或者想要与其他mod进行交互，那么建议你在你的API中放置你的`IProperty`（和任何用作值的类）。 这样，其他人可以使用属性和值来设置世界中的块，而不必像以前那样使用任意数字。

在创建了`IProperty<>`对象之后，在Block类中重写`createBlockState`方法。 在该方法中，只需写`return new BlockState()`。 首先将`BlockState`用构造函数传递给你的Block，`this`，然后在它下面写你要声明的`IProperty`。 请注意，在1.9及更高版本中，`BlockState`类已重命名为`BlockStateContainer`，更符合此类实际执行的操作。

你刚创建的对象是一个非常神奇的对象 - 它管理着上面所有三元组的生成。 也就是说，它为每个属性生成每个值的所有可能组合（对于学过数学的人，它采用每个属性的可能值集合并计算这些集合的笛卡尔积）。 因此，给定任意的属性`IBlockState`它可以生成对应的（块，属性，值）。

如果您没有将其中一个`IBlockState`设置为您的Block的默认状态，那么将自动为您选择一个。 你可能不希望这样（因为在大多数情况下它会导致奇怪的事情发生），所以在你的Block的构造函数调用`setDefaultState()`结束时，传入你想成为默认值的`IBlockState`。 使用`this.blockState.getBaseState()`获取为您选择的那个，然后使用`withProperty`为*每个*属性设置一个值。

因为`IBlockState`是不可变的预生成的，所以调用`IBlockState.withProperty(<PROPERTY>，<NEW_VALUE>)`将简单地转到`BlockState` /`BlockStateContainer`并使用你想要的值集请求IBlockState， 而不是构建一个新的“IBlockState”。

由此可以很容易地得出，因为基本的`IBlockState`是在启动时生成固定集的，所以你能够并且鼓励使用引用比较（==）来检查它们是否相等！


使用`IBlockState`
---------------------
正如你现在所知，`IBlockState`是一个强大的对象。 你可以通过调用，`getValue(<PROPERTY>)`传递你要测试的`IProperty<>`来获取属性的值。
如果你想获得一个具有不同值的IBlockState，只需调用上面提到的`withProperty(<PROPERTY>, <NEW_VALUE>)`。 这将使用您请求的值返回另一个预生成的`IBlockState`。

你可以使用`setBlockState()`和`getBlockState()`来获取和放置`IBlockState`。


幻想杀手
----------------
遗憾的是，抽象是他们的核心所在。 我们仍然有责任将每个`IBlockState`转换回0到15之间的数字，这些数字将存储在世界中，反之亦然，以便加载。

如果你声明任何`IProperty`，你**必须**重写`getMetaFromState`和`getStateFromMeta`

在这里，您将读取属性的值并返回0到15之间的适当整数，或者反过来; 读者可以自己查看原本的例子。

!!! Warning "警告"
    你的 `getMetaFromState` 和 `getStateFromMeta` 方法必须是一对一的！换句话说，相同的属性与值必须映射到同一meta值，反之亦然。如果你没有做到这个，游戏将不会崩溃。它只会让所有东西都表现得很奇怪。


“实际”状态 
-------------
你可能会注意到栅栏并不将连接属性记录到meta中，但在F3菜单中你仍可以看到它的属性和值！这是为什么呢？

方块可以声明不保存到Metadata中的属性。这些通常用作渲染目的，但也可以有一些其他的用处。

你仍可以在 `createBlockState` 中声明它们，并且使用 `setDefaultState` 设置值。然而，这些值在 `getMetaFromState` 和 `getStateFromMeta` 里都**不要**动。

然而，你需要在你的 `Block` 类中重写 `getActualState`。这里你将会接收到对应世界中Metadata值的 `IBlockState` ，你需要返回另一个包含缺失信息的 `IBlockState`，比如说栅栏链接，红石链接等。你需要用 `withProperty` 来附加属性。你也可以用这个来对一个值读TileEntity数据（当然你需要恰当的安全检查！）。

!!! warning "警告"

    当你在 `getActualState` 中读取TileEntity的数据时，你必须采取进一步的安全检查。默认情况下，`getTileEntity` 将会在TileEntity不存在时尝试创建这个TileEntity。然而，`getActualState` 和 `getExtendedState` 可以并且会被另外的线程调用，这将会导致在它尝试创建丢失TIleEntity时，世界的TileEntity列表会抛出 `ConcurrentModificationException`。所以，你必须检查 `IBlockAccess` 参数是否为 `ChunkCache`（传给其它线程的对象），如果是的话，将其强制转换为 `ChunkCache` 类型，并使用 `getTileEntity` 不可写的变种。安全检查的例子可以在 `BlockFlowerPot.getActualState()` 中找到。

!!! note "提示"

	调用 `world.getBlockState()` 将会返回代表metadata的 `IBlockState`。所以返回的 `IBlockState` 将不会包含 `getActualState` 内的数据。如果这写数据对你的代码很重要，请务必调用 `getActualState`！

[拓展状态](../models/advanced/extended-blockstates.md)
------------------------
扩展的方块状态是一种在呈现方块间将任意数据传递到`IBakedModel`的方法。 它们主要用于渲染的上下文中，因此在“模型”部分中有记录。
