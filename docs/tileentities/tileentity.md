# TileEntities

Tile Entities就像简化的实体一样，绑定到Block上。
它们用于存储动态数据，执行基于tick的任务以及动态渲染。
原本 Minecraft的一些例子是：处理库存（箱子），熔炉上的冶炼逻辑或信标的区域效应。
mod中存在更高级的示例，例如采石场，分拣机，管道和显示器。

!!! note "提示"

    `TileEntities`不是一切的解决方案，如果使用不当会导致卡顿。
     请尽可能不要使用它们。

## 创建 `TileEntity`

为了创建`TileEntity`，你需要继承`TileEntity`类。
重要的是你的`TileEntity`有一个默认的构造函数，以便Minecraft可以正确加载它。
创建类后，需要注册`TileEntity`。 为此你需要调用：

```JAVA
GameRegistry#registerTileEntity(Class<? extends TileEntity> tileEntityClass, ResourceLocation key)
```

第一个参数是你的TileEntity类，第二个参数是你的TileEntity的注册表名称。
此时，您可以选择在`FMLPreInitializationEvent`期间或在`RegistryEvent.Register <Block>`事件期间执行此操作，因为`TileEntities`还没有自己的注册表事件。

!!! note "提示"

    在Forge版本`14.23.3.2694`之前注册`TileEntity`的方法是使用`String`而不是`ResourceLocation`。
    此方法从String创建`ResourceLocation`。 一定要使用ResourceLocation格式modid:tile_entity来避免它被改为minecraft:tile_entity。

## `TileEntity` 连接到方块上

要将新的`TileEntity`附加到`Block`，您需要覆盖Block类中的2个方法。

```JAVA
Block#hasTileEntity(IBlockstate state)

Block#createTileEntity(World world, IBlockState state)
```
使用这些参数，您可以选择块是否应该具有`TileEntity`。
通常，您将在第一个方法中返回`true`，在第二个方法中返回`TileEntity`的新实例。

## <a id="store">用`TileEntity`储存数据</a>

可以重写这两个方法来储存数据：
```JAVA
TileEntity#writeToNBT(NBTTagCompound nbt)

TileEntity#readFromNBT(NBTTagCompound nbt)
```
每当包含`TileEntity`的区块加载/保存NBT数据时，就会调用这些方法。使用它们来读取和写入TileEntity类中的字段。

!!! note "提示"

    每当您的数据发生变化时，您需要调用`TileEntity＃markDirty()`，否则在保存世界时可能会跳过包含您的`TileEntity`的区块。

!!! important "重要"

    调用父类方法很重要！
    标签名称`id`，`x`，`y`，`z`，`ForgeData`和`ForgeCaps`由父类方法保留。

## 通过 `BlockStates`保存 `TileEntity` 

可能存在需要更改`BlockState`的情况，例如使用原本熔炉，当燃料和内部物品燃烧时，原本熔炉将其状态从`lit=false`改为`lit=true`。
通过覆盖以下方法实，现这一点非常简单：

```JAVA
TileEntity#shouldRefresh(World world, BlockPos pos, IBlockState oldState, IBlockState newSate)
```

!!! important "重要"

    你应该检查`BlockStates`而不仅仅是返回`false`,而要以防止不必要的行为和错误。 特别是当你的`Block(State)`被另一个取代时,比如说`Air`，也会调用这个方法。

## 刷新`TileEntities`

如果您需要刷新`TileEntity`，例如为了刷新冶炼过程中的进度，您的`TileEntity`需要实现`net.minecraft.util.ITickable`接口。
然后，您可以在其中实现所有计算：

```JAVA
ITickable#update()
```

!!! note "提示"

    每个tick都会调用此方法，因此您应该避免在此处进行复杂的计算。
    如果可能的话，您应该在每个X tick处进行更复杂的计算。
    （一秒钟内的tick数可能低于20但不会更高）

## 将数据同步到客户端

有三种方法可以将数据同步到客户端。
同步区块加载，同步区块更新并与自定义网络消息同步。

### 同步区块加载

首先你需要重写：
```JAVA
TileEntity#getUpdateTag()

TileEntity#handleUpdateTag(NBTTagCompound nbt)
```

同样，这很简单，第一种方法收集应该发送给客户端的数据，而第二个处理该数据。 如果您的`TileEntity`不包含太多数据，您可以使用[在TileEntity中存储数据][Storing]部分中的方法。

!!! important "重要"

    为TileEntities同步过多或无用的数据可能导致网络拥塞。 您应该通过仅在客户端需要时发送客户端所需的信息来优化您的网络使用。 例如，通常不需要在更新标签中发送TileEntity，因为这可以通过GUI同步。

### 同步方块更新

这个方法有点复杂，但你只需要重写2个方法。
这是一个很小的示例：

```JAVA
@Override
public SPacketUpdateTileEntity getUpdatePacket(){
    NBTTagCompound nbtTag = new NBTTagCompound();
    //在这里写入NBT数据
    return new SPacketUpdateTileEntity(getPos(), 1, nbtTag);
}

@Override
public void onDataPacket(NetworkManager net, SPacketUpdateTileEntity pkt){
    NBTTagCompound tag = pkt.getNbtCompound();
    //处理你的数据
}
```
`SPacketUpdateTileEntity`的构造函数需要：

* `TileEntity`的位置.
* 一个ID，虽然除了原本之外它并没有真正用过，因此你可以在那里放一个1。
* 一个包含你的数据的`NBTTagCompound`

除此之外，您现在需要在客户端上生成"BlockUpdate”。

```JAVA
World#notifyBlockUpdate(BlockPos pos, IBlockState oldState, IBlockState newState, int flags)
```
`pos`是你的TileEntitiy的位置。 对于`oldState`和`newState`，您可以传递当前的BlockState。

`flags`是一个位掩码，应该包含2，它将把更改同步到客户端。

### 使用自定义网络消息进行同步

这种同步方式可能是最复杂的方式，但通常也是最优化的方式，因为您可以确保只同步你需要数据。
在尝试此操作之前，您应首先查看[网络][networking]部分，特别是[SimpleImpl][simple_impl]。
创建自定义网络消息后，您可以将其发送给加载了“TileEntity”的所有用户：

```JAVA
SimpleNetworkWrapper#sendToAllTracking(IMessage, NetworkRegistry.TargetPoint)
```

!!! warning "警告"

    重要的是你进行安全检查，当消息到达玩家时，TileEntity可能已被销毁/替换！
    你还应该检查是否加载了区块（`World＃isBlockLoaded(BlockPos)`）

[networking]: ../networking/index.md
[simple_impl]: ../networking/simpleimpl.md
[Storing]: 	#store

