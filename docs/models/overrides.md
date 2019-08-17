物品属性概述
=======================

物品属性是一种通过“属性”设定模型的方式。弓就是一个例子，它最重要的一个属性是它拉了多远。这个信息用于决定它用哪个模型，做出拉动的动画。与直接通过`ModelLoader.setCustomModelResourceLocation` 或 `ModelLoader.setCustomMeshDefinition`给物品分配`ModelResourceLocation`不同。这些方法固定了可能的模型集。例如弓，这些方法会固定拉动动画中的帧数为4，然而，属性是可变的。

物品属性给每个注册的`ItemStack`分配一个确定的`float`，且原版模型的定义可以用这些值来“覆盖”默认的模型，如果覆盖匹配，它会覆盖并使用另一个。物品模型的格式和覆盖可以在[wiki][format]上找到。这很有用因为它是连续的。例如，弓用物品属性来定义它的拉动动画。由于属性的值是一个`float`，它在0到1上连续增加。这可以让材质包根据想要的增幅增加许多模型，而不是限制在默认动画中的4个模型。指南针和钟也同样如此。

给物品添加属性
--------------------------

用`Item::addPropertyOverride`给物品添加属性。`ResourceLocation`参数是给属性的名字(例如`new ResourceLocation("pull")`)。`IItemPropertyGetter`是一个回调函数，其中参数有拿着的`ItemStack`，所在的`World`，拿着它的实体生物`EntityLivingBase`，该回调函数返回`float`类型，即物品的属性。一些列子是`ItemBow`里的"`pulling`"和"`pull`"属性，和`Item`里的几个`static final`的属性。对于mod里的属性，推荐用modid作为命名空间（例如`examplemod:property`而不仅仅是`property`，因为那样实际上是`minecraft:property`）。

Using Overrides
---------------

覆盖的格式可以参考[wiki][format]，`model/item/bow.json`是一个很好的例子。作为参考，这是一个假想的例子，有一个有`examplemod:power`属性的物品。如果它的值不匹配，则是默认模型。

!!! important "重要"

    predicate匹配 *大于等于* 给定值的值。

```json
{
  "parent": "item/generated",
  "textures": {
    "__comment": "默认",
    "layer0": "examplemod:items/examplePartial"
  },
  "overrides": [
    {
      "__comment": "power >= .75 时",
      "predicate": {
        "examplemod:power": 0.75
      },
      "model": "examplemod:item/examplePowered"
    }
  ]
}
```

这是配合它的一段代码。(它不一定要仅客户端(client-only)；它也可以在服务端工作。在原版里，属性在物品的构造器中注册。)

```java
item.addPropertyOverride(new IItemPropertyGetter() {
  @SideOnly(Side.CLIENT)
  @Override
  public float apply(ItemStack stack, @Nullable World world, @Nullable EntityLivingBase entity) {
    return (float)getPowerLevel(stack) / (float)getMaxPower(stack); // 一些其它代码
  }
}
```

[format]: https://minecraft-zh.gamepedia.com/%E6%A8%A1%E5%9E%8B#%E7%89%A9%E5%93%81%E6%A8%A1%E5%9E%8B
