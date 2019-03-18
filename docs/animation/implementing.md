使用 API
======================

根据您希望使用API制作动画不同，代码端实现有点不同。
有关ASM API本身（用于控制动画）的文档可在[ASM][asm]页面上找到，因为它与您动画的内容无关。

方块
--------

方块的动画是用`AnimationTESR`完成的，它是一个`FastTESR`。 因此，必须为您的块设置`TileEntity`。 你的`TileEntity`必须提供`ANIMATION_CAPABILITY`，它通过你的ASM调用它的`.cast`方法来接收。 如果你没有在方块的块状态中提供`StaticProperty`，那么你的方块也必须在`ENTITYBLOCK_ANIMATED`渲染层中渲染。

`StaticProperty`是一个属性，你可以通过在`createBlockState()`里面的方块的属性列表中添加`Properties.StaticProperty`来添加到方块的块状态。 渲染方块时，`AnimationTESR`检查属性的值是否为真; 如果为真，则块渲染继续正常进行。 否则，`AnimationTESR`动画分配给方块状态json中的`static=false`变体的方块模型。 模型所有的静态部分应该静态渲染的，因为这是它的目的。

`handleEvents()`回调位于`AnimationTESR` 中，因此在注册`TileEntity`时必须重载。

这是一个注册TESR的示例:

```java
ClientRegistry.bindTileEntitySpecialRenderer(Chest.class, new AnimationTESR<Chest>()
{
    @Override
    public void handleEvents(Chest chest, float time, Iterable<Event> pastEvents)
    {
        chest.handleEvents(time, pastEvents);
    }
}); 
```

在这个例子中，我们在注册TESR时重写了`handleEvents()`回调，因为实现很简单，但你可以轻松地继承`AnimationTESR`以实现相同的效果。 方块的`handleEvents()`回调有两个参数：正在渲染的tile实体和一个可迭代的事件。
对`chest.handleEvents()`的调用调用位于虚构的`Chest`TileEntity中的方法，因为在`handleEvents()`方法中无法访问ASM。

物品
-------

物品的动画完全使用功能系统完成。 您的物品必须通过`ICapabilityProvider`提供`ANIMATION_CAPABILITY`。 你可以创建这个功能的一个实例，它使用你的ASM的`.cast`方法，它通常存储在`ICapabilityProvider`对象本身。 下面是一个例子：

```java
private static class ItemAnimationHolder implements ICapabilityProvider
{
    private final VariableValue cycleLength = new VariableValue(4);

    private final IAnimationStateMachine asm = proxy.load(new ResourceLocation(MODID.toLowerCase(), "asms/block/engine.json"), ImmutableMap.<String, ITimeValue>of(
        "cycle_length", cycleLength
    ));

    @Override
    public boolean hasCapability(@Nonnull Capability<?> capability, @Nullable EnumFacing facing)
    {
        return capability == CapabilityAnimation.ANIMATION_CAPABILITY;
    }

    @Override
    @Nullable
    public <T> T getCapability(@Nonnull Capability<T> capability, @Nullable EnumFacing facing)
    {
        if(capability == CapabilityAnimation.ANIMATION_CAPABILITY)
        {
            return CapabilityAnimation.ANIMATION_CAPABILITY.cast(asm);
        }
        return null;
    }
}
```

无法在当前实现中的物品上接收事件。

实体
----------

为了使用动画API为实体设置动画，实体的渲染器必须将`AnimationModelBase`作为其模型。 该模型的构造函数采用两个参数，即实际模型的位置（如JSON或B3D文件的路径，而不是方块状态引用）和`VertexLighter`。
可以使用`new VertexLighterSmoothAo(Minecraft.getMinecraft().getBlockColors())`创建`VertexLighter`对象。
实体还必须提供`ANIMATION_CAPABILITY`，它可以通过传递ASM以`.cast`方法创建。

`handleEvents()`回调位于`AnimationModelBase`类中，如果你想使用该事件，你必须继承`AnimationModelBase`类。 回调有三个参数：正在渲染的实体，当前时间(tick)，以及已发生事件的可迭代对象。

创建渲染器的示例如下所示:

```java
ResourceLocation location = new ModelResourceLocation(new ResourceLocation(MODID, blockName), "entity");
return new RenderLiving<EntityChest>(manager, new net.minecraftforge.client.model.animation.AnimationModelBase<EntityChest>(location, new VertexLighterSmoothAo(Minecraft.getMinecraft().getBlockColors()))
{
    @Override
    public void handleEvents(EntityChest chest, float time, Iterable<Event> pastEvents)
    {
        chest.handleEvents(time, pastEvents);
    }
}, 0.5f) {

// ... getEntityTexture() ...

};
```

[asm]: asm.md
