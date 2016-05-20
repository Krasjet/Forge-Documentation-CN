方块状态(Block States)
=====================

请在开始写代码前将本指南**全部**阅读完。你的理解将会比你阅读部分只更加全面和正确。

这个指南旨在对方块状态(Block States)提供一个入门等级的介绍。如果你知道什么是拓展状态(Extended States)，你可能会注意到我精简了一下东西。这是有意的，我们不想给初学者太多不会及时用到的信息。如果你不知道什么是拓展状态，不要担心，我们在之后会另外写一个文档。

使用方块状态的原因
================

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

与其到处用一些意义不明的数字，我们选择声明方块需要包含的**属性(Properties)**。自然，这个接口的名字为 `IProperty`。  
注意Metadata并没有移除，新的系统只是允许我们将所有难看的数字塞到两个方法离去，之后忘掉它们。

在你的方块类，重写(Override) `createBlockState()` 这个方法。它将会在方块第一次初始化的时候被调用，并返回一个 `BlockState` 对象。这个名字起得不太好。它不是一个方块的状态，他只是管理了你方块的所有方块状态。它的名字在1.9中被改为了更恰当的 `BlockStateContainer`。调用这个类的构造器(Constructor)，并将你方块的对象传过去，还有方块所需的所有 `IProperty`。`IProperty` 需要是 static final字段。Minecraft原版中将它们储存在对应的方块类中，在其他mod中可能很多方块共用一个 `IProperty`。

原版提供给我们几个 `IProperty<?>` 的实现：

- `PropertyInteger`：实现 `IProperty<Integer>`。通过调用 `PropertyInteger.create("<name>", <min>, <max>);` 来创建。
- `PropertyBool`：实现 `IProperty<Boolean>`。通过调用`PropertyBool.create("<name>");` 来创建
- `PropertyEnum<E extends Enum<E>>`：实现 `IProperty<E>`，定义了一个包含Enum类值的属性。通过调用 `PropertyEnum.create("name", <enum_class>);` 来创建
  - 你也可以通过可变参数或者Collection指定特定的有效枚举值。有一个Factory可以在值有效的时候返回true
- `PropertyDirection`：这是 `PropertyEnum<EnumFacing>` 的一个简便实现
  - 一些简便的断言(Predicate)也有提供。比如说，如果要获取一个表示主方向的属性，你可以调用 `PropertyDirection.create("<name>", EnumFacing.Plane.HORIZONTAL)`。如果你需要获取X方向，调用 `PropertyDirection.create("<name>", EnumFacing.Axis.X)`

最终，Minecraft将会通过生成所有可能的属性值组合为你的方块生成所有可能的 `IBlockState` 对象。

!!! note

	如果你的mod有一个API，或者需要与其他mod进行互动。我**非常非常**建议你将你的 `IProperty` 储存在你的API中。这样的话用户就可以使用方块状态来改变你的方块，而不是用一些意义不明的Metadata。

`IBlockState`
-------------

`IBlockState` 是1.8中新的 "方块 + meta"。它包含了一个 `Block` 的信息，它的属性，和每个属性对应的**值**。比如说，一个木头的 `IBlockState` 包含 `Block` "Blocks.log" 或者 "Blocks.log2"，属性 AXIS和VARIANT，和这些属性的**值**，例如 `Axis.Y` 和 `BlockPlanks.WoodType.BIRCH`。  
你可以通过 `world.getBlockState(BlockPos)` 来获取。  
要想检测一个属性的值，使用 `IBlockState.getValue(<property>)`。所以对我们的木头来说，如果我们想检测其是否为桦木，我们可以使用 `world.getBlockState(pos).getValue(VARIANT) == BlockPlanks.WoodType.BIRCH`。  
要想获取其它属性组合的 `IBlockState` 对象，用 `withProperty(IProperty<T>, T)`。它将会返回一个包含你所指定值的不同 `IBlockState` 对象。

!!! note

	所有的 `IBlockState` 对象都由Minecraft在启动的时候生成的。对于每一个特定的 "方块 + 属性 + 值"，只存在一个 `IBlockState`。所以通过引用相等(==)来比较 `IBlockState` 是完全安全并且推荐的。

在你方块的构造器(Constructor)中，你需要设定默认的方块状态。你可以通过调用 `setDefaultState(blockState.getBaseState().withProperty(...))` 来实现。记得一定要包含你**所有的** `IProperty`，否则你忽略掉的值将会在默认方块状态中为未定义状态，它将导致一些不会导致游戏崩溃但会造成奇怪行为的隐性bug。

从Meta保存和读取
---------------

如果你的方块没有使用 `IProperty` ，那么你不需要重写这两个方法。如果你使用了，那你**必须**要重写，否则Minecraft将会在启动的时候崩溃。  
重写 `getMetaFromState` 和 `getStateFromMeta`。你需要在这两个方法中进行位掩码，位压缩，及位检测等操作。注意由于保存格式没有改变，你仍然只能有16个meta值(0-15)。

!!! warning

	你的 `getMetaFromState` 和 `getStateFromMeta` 方法**必须**是一对一的！换句话说，相同的属性与值必须映射到同一meta值，反之亦然。如果你没有做到这个，游戏将**不会**崩溃。它只会让所有东西都表现得很奇怪。

`IBlockState`：不仅仅是Block和Meta
---------------------------------

这并不是故事的结束。  
如果你指向一个原版的栅栏并将F3打开，你可能会注意到boolean属性north，south，west，和east。你可能会注意到栅栏并不将连接属性记录到meta中，这是为什么呢？  
显然，`IBlockState` 可以储存没有保存到meta的属性的值。你可以同样在 `createBlockState` 中注册他们。  
在你方块类中重写 `getActualState`。这里，栅栏和红石检测四周的连接，并相应修改方块的状态。

!!! note

	调用 `world.getBlockState()` 将会返回代表metadata的 `IBlockState`。所以返回的 `IBlockState` 将不会包含 `getActualState` 内的数据。如果这写数据对你的代码很重要，请务必调用 `getActualState`！

!!! warning

	在 `getMetaFromState`/`getStateFromMeta` **不要**动至于渲染相关的属性！这两个方法只应该操作你需要储存的属性。

拓展阅读
-------

1.8中的渲染和拓展状态：未完成