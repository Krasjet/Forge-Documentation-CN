彩色纹理
=================

原版中的许多方块和物品(如草方块)会根据它们的位置改变其纹理颜色。 模型支持在面上指定“色调指数”，这些数字可以由`IBlockColor`和`IItemColor`处理。 有关如何在原版模型中定义色调指数的信息，请参阅[wiki][]。

### `IBlockColor`/`IItemColor`

这两个都是单方法接口。 `IBlockColor`需要一个`IBlockState`，一个（可为null）的`IBlockAccess`和一个（可为null）的`BlockPos`。 `IItemColor`需要一个`ItemStack`。它们都需要参数`tintindex`，这是被着色的面的色调指数。它们都返回一个`int`，一个颜色乘数。这个`int`按顺序被视为4个无符号字节，透明度，红，绿和蓝，从最高的字节到最低字节。对于着色面中的每个像素，每个颜色通道的值是`(int)((float)base * multiplier / 255)`，其中`base`是通道的原始值，`multiplier`是关联的来自颜色乘数的字节。请注意，方块不使用alpha通道。例如，草纹理，未着色，看起来是白色和灰色的。用于草的`IBlockColor`和`IItemColor`返回颜色乘数，具有低红色和蓝色成分，但是高透明度和绿色成分，因此当执行乘法时，绿色被带出并且红色/蓝色减少了。

如果物品继承自`builtin/generated`模型，则每个图层（“layer0”，“layer1”等）都具有与其图层索引对应的色调指数。

### 创建颜色控制器

`IBlockColors`需要注册到游戏的`BlockColors`实例。 `BlockColors`可以通过`Minecraft.getMinecraft().getBlockColors()`获得，`IBlockColor`可以通过`BlockColors::registerBlockColorHandler`注册。 请注意，这不会导致给定方块的`ItemBlock`被着色。 `ItemBlock`是物品，需要用`IItemColor`着色。

`IItemColors`需要注册到游戏的`ItemColors`实例。 `ItemColors`可以通过`Minecraft.getMinecraft().getItemColors()`获得，`IItemColor`可以通过`ItemColors::registerItemColorHandler`注册。 这个方法被重载也需要`Block`，它只是为物品的`Item.getItemFromBlock（block）`注册颜色控制器（即方块的`ItemBlock`）。

注册只能在初始化阶段在客户端完成.

[wiki]: https://minecraft.gamepedia.com/Model#Block_models
