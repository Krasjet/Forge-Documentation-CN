物品
====

和方块一样，物品也是大部分Mod的关键组成部分之一。方块组成你身边的世界，而物品则能够让你改变世界。

创建物品
-------

### 简单物品

没有特殊功能的简单物品（比如木棍或者糖）不需要自定义的类。你可以直接实例化一个 `Item`，再调用Setter去设置物品一些简单的属性。

|         方法         |                  描述                  |
|:----------------------:|:----------------------------------------------|
|    `setCreativeTab`    | 设置物品所在的创造标签页。如果你想让这个物品在创造模式菜单中显示，这个方法必须要调用。原版的标签页能在 `CreativeTabs` 类中找到 |
|     `setMaxDamage`     | 设置物品的最大损害值(Damage Value)。如果这个值大于 `0`，将会添加2个物品属性 "damaged" 和 "damage" |
|    `setMaxStackSize`   | 设置最大的堆(Stack)大小                  |
|      `setNoRepair`     | 让这个物品无法修复，即使它是可损害的(Damagable) |
|  `setUnlocalizedName`  | 设置物品的未本地化名，有 "item." 前缀 |
|    `setHarvestLevel`   | 添加或者移除一对挖掘(Harvest)类别（`"shovel"`，`"axe"`）和挖掘等级。这个方法是不可链(Not Chainable)的 |

除非特殊说明，上面的方法都是可链的(Chainable)，也就是说它们会 `return this` 从而让你能够连续调用它们。

### 高级物品

上面设置属性的方法只能够用于简单的物品。如果你希望有一个更复杂的物品，你应该创建一个 `Item` 的子类并重写它的方法。

注册物品
-------

物品必须要[注册][registering]到函数中。

为物品着色
---------

物体的纹理是可以在程序中着色的。许多原版的物品都用到了这一功能。比如说：皮革帽子、生成蛋、药水等。

### 物品颜色处理器

物体颜色处理器是对一个物体着色所必须的东西，它是`IItemColor`的一个实例，它添加了以下方法：

```java
int getColorFromItemstack(
  ItemStack stack, 
  int tintIndex)
```

#### 返回值

这个方法返回一个十六进制值(Hex)，用整数表示一个颜色。

#### 参数

色调指数(Tint Index)是定义在物品模型JSON文件中的一个元素的面中的。没有色调指数的面是不会被着色的，所以它的颜色处理器将不会被调用。当物品的模型是从`builtin/generted`继承而来时，层指数会被用作色调指数。

### 注册物体颜色处理器

物体颜色处理器必须要通过调用`ItemColors#registerItemColorHandler(IItemColor, Item...)`函数来注册。`ItemColors`的实例可以通过调用`Minecraft#getItemColors()`来获取。

这必须要在初始化阶段中，仅在客户端调用。

[registering]: ../concepts/registries.md#_2