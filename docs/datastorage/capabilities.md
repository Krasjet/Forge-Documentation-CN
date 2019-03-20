能力系统
========

能力(Capability)使特性更动态和灵活地展现(Expose)，而不必直接实现很多接口(Interface)。

总的来说，每个能力以接口形式提供了一个特性，一个可被调用的默认实现，和一个至少对于默认实现的储存处理器(Storage Handler)。储存管理器可以支持其它的实现，但是这个是取决于能力的实现者，所以你应该先看一下它们的文档之后再决定是否对非默认实现使用默认储存。

Forge对TileEntity、实体、ItemStack、世界和区块添加了能力支持。它们可以通过事件附加上去，也可以通过在你的实现中重写能力的方法来展现。这将在后面的小节中进行更详细地解释。

Forge提供的能力
--------------

Forge提供了三种能力：IItemHandler、IFluidHandler和IEnergyStorage。

IItemHandler展现了一个处理背包格子的接口。它可以被应用到TileEntity（箱子，机器）、实体（玩家更多的背包格子，生物的背包）、或ItemStack（便携背包等）。它通过一个自动化友好的系统代替了以前的 `IInventory` 和 `ISidedInventory`。

IFluidHandler展现了一个处理液体存储的接口。它可以被应用到TileEntity、实体或ItemStack中。它通过一个更一致和自动化友好的系统代替了以前的`IFluidHandler`

IEnergyStorage展现了一个用于处理能量容器的接口。它可以被应用到TileEntity、实体或ItemStack中。这个能力是基于TeamCoFH的RedstoneFlux API而制作的。

使用现有能力
-----------

正如之前提到的那样，TileEntity、实体、和ItemStack通过 `ICapabilityProvider` 实现了能力Provider特性。这一接口添加了两个方法：`hasCapability` 和 `getCapability`，它们可以用来获取对象中存在的能力。

要想获取能力，你首先要用其唯一的实例(Instance)引用它。在这里是Item Handler，这个能力一般储存在 `CapabilityItemHandler.ITEM_HANDLER_CAPABILITY`，但是我们也可以用 `@CapabilityInject` 这个注解(Annotation)获取其它的实例。

```java
@CapabilityInject(IItemHandler.class)
static Capability<IItemHandler> ITEM_HANDLER_CAPABILITY = null;
```

这个注解可以被应用在字段(Field)和方法(Method)上。当应用到字段上的时候，它会在能力注册时将能力的实例（同一个实例赋值到所有字段中）赋值到字段上，如果该能力没有被注册，它将保持现有值(`null`)。因为本地静态字段访问会更快，所以您最好保留一份能力对象的引用。这个注解也可以同样应用到方法中，应用的方法将在能力注册时被调用，以便某些特性的应用。

`hasCapability` 和 `getCapability` 这两个方法都有第二个参数，类型为 `EnumFacing`，他们可以用来对特定面请求特定的实例。如果传入的是 `null`，那么就可以认定请求是从方块内或者从一个与面无意义的地方来的，比如说另一个维度(Dimension)。这时候一个不考虑面的通用能力实例将会被请求。`getCapability` 的返回类型将会对应能力声明时的类型。对于Item Handler这个能力，它是 `IItemHandler`。

展现一个能力
-----------

要想展现(Expose)一个能力，你需要先获得一个潜在能力类型的实例。注意你需要对每个存有能力的对象赋值不同的实例，因为能力很可能会和包含的对象联系起来。

有两种方式能获得实例，第一种是通过 `Capability` 本身，第二种是显式实例化一个它的实现。第一种方法是为使用默认实现设计的，如果那些默认的值对你很有用的话，你可以选择使用它。Item Handler这个能力的默认实现只会展现一个单格背包(Inventory)，可能不是你想要的结果。

第二种方法可以用自定义的实现。在 `IItemHandler` 这个例子中，默认实现使用的是 `ItemStackHandler` 类，它有一个可选的带参数的构造器，参数为格子的个数。然而，我们不应认定默认实现是一直存在的，因为能力系统设计初衷就是为了防止能力不存在时的加载错误。所以实例化之前我们需要检查能力是否被注册了（见上面关于 `@CapabilityInject` 的内容）。

一旦你有了自己的能力接口实例，你会希望在你展现能力的时候通知能力系统的用户。这个可以通过重写 `hasCapability` 方法进行实现，并将实例和你展示的能力进行对比。如果你的机器根据面的不同有不同数量的格子，你可以通过 `facing` 参数进行判定。对于实体和ItemStack，这个参数可以忽略掉，但是你仍然可以自己对面进行定义以使用这一参数，比如说玩家不同的装备格（顶面 => 头部格子？），或者背包中四周的格子（西面 => 左边的格子？）。不要忘记回溯到 `super` 中，否则附加的能力将会停止工作。

```java
@Override
public boolean hasCapability(Capability<?> capability, EnumFacing facing) {
  if (capability == CapabilityItemHandler.ITEM_HANDLER_CAPABILITY) {
    return true;
  }
  return super.hasCapability(capability, facing);
}
```

同样，当接收到请求的时候，你需要提供能力实例的接口引用。一样，不要忘记回溯到 `super`。

```java
@Override
public <T> T getCapability(Capability<T> capability, EnumFacing facing) {
  if (capability == CapabilityItemHandler.ITEM_HANDLER_CAPABILITY) {
    return (T) inventory;
  }
  return super.getCapability(capability, facing);
}
```

我们强烈建议您使用直接检查来判定能力而不是尝试依靠于地图和其他数据结构，因为能力判定每个tick可能对多个对象进行，它们需要以最快速度完成以避免游戏卡顿。

附加能力
-------

之前说过，对实体和ItemStack附加能力可以通过 `AttachCapabilityEvent` 来完成。所有提供了能力的对象使用的都是这同一个事件。`AttachCapabilityEvent` 有五个泛型(Generic)，分别提供了以下几个事件

- `AttachCapabilityEvent<Entity>`: 仅对实体触发
- `AttachCapabilityEvent<TileEntity>`: 仅对TileEntity触发
- `AttachCapabilityEvent<Item>`: 仅对ItemStack触发
- `AttachCapabilityEvent<World>`: 仅对世界触发
- `AttachCapabilityEvent<Chunk>`：仅对区块触发

泛型的类型只能是以上几个，不能够更加细化。比如说，如果你想附加一个能力到 `EntityPlayer` 上，你必须要订阅的是 `AttachCapabilityEvent<Entity>`，之后在附加相应能力之前判断所提供的对象是否是 `EntityPlayer`。

每个事件都有一个方法 `addCapacity`，它可以用来附加能力到目标对象上。你在能力列表中添加的是能力Provider，而不是能力本身，它可以从特定面返回相应的能力。Provider只需要实现 `ICapabilityProvider`，如果能力需要持久储存数据，你需要实现 `ICapabilitySerializable<T extends NBTBase>`，它不仅能返回能力，还能提供NBT存储与读取函数。

要想获得实现 `ICapabilityProvider` 的更多信息，请看上面的[展现一个能力](#_2)小节。

创建你自己的能力
--------------

一般来说，一个能力通过一个单独的 `CapabilityManager.INSTANCE.register()` 方法调用即可同时声明及注册。还有一种可能是在能力单独的一个类中定义一个静态的 `register()` 方法，但这个能力系统并不做要求。在这个文档中，我们将会将每一个部分都分成单独的命名类，尽管你也可以使用匿名类。

```java
CapabilityManager.INSTANCE.register(能力接口类, storage, 默认实现Factory);
```

第一个参数是描述能力特性的类型，在这个例子中是 `IExampleCapability.class`。

第二个参数是一个实现 `Capability.IStorage<T>` 的类实例，T是第一个参数中的类。这个储存(Storage)类将帮助我们对默认实现进行储存和读取管理，它也可以支持其它的实现

```java
private static class Storage
    implements Capability.IStorage<IExampleCapability> {

  @Override
  public NBTBase writeNBT(Capability<IExampleCapability> capability, IExampleCapability instance, EnumFacing side) {
    // 返回一个NBT Tag
  }

  @Override
  public void readNBT(Capability<IExampleCapability> capability, IExampleCapability instance, EnumFacing side, NBTBase nbt) {
    // 从NBT Tag读取
  }
}
```

最后一个参数是一个可调用的Factory，它会返回默认实现的新建实例。

```java
private static class Factory implements Callable<IExampleCapability> {

  @Override
  public IExampleCapability call() throws Exception {
    return new Implementation();
  }
}
```

最后我们需要一个默认实现，以能在Factory中实例化。这个类的设计完全取决于你，但它至少应该能提供一个简单的骨架让别人测试这个能力（如果它本身不是完全可用的话）。

!!! warning "警告"

	和其它拥有能力的对象不同，区块只会在该区块被标记(Mark Dirty)的时候才会写入磁盘。`Chunk`的能力实现必须要保证在改变状态时，对区块进行标记。


与客户端同步数据
--------------

默认情况下，能力数据将不会被发送到客户端。为了修复这个问题，Mod需要用网络数据包(Packet)进行自己的同步。

对于下面三种可选的情况你可能会想发送同步数据包：

1. 当实体生成在世界时，你可能需要与客户端同步初始值
2. 当储存的数据改变时，你可能需要通知一些看到实体的客户端
3. 当一个新的客户端开始看到实体，你可能需要提醒它现有的数据

如果你想要更多网络数据包的实现和信息，请去看[网络](../networking/index.md)部分的教程。

在玩家死亡时保持数据
------------------

默认情况下，实体的能力数据在死亡的时候就会消失。为了改变这一点，在玩家实体重生被克隆时数据需要被复制下来。

这个可以通过处理 `PlayerEvent.Clone` 这个事件来完成。这个事件中，`wasDead` 这个字段可以用来判别是死亡后的重生还是从末地之路返回到主世界。这个检测很重要，因为如果是从末地返回的话数据仍然是存在的。

从IExtendedEntityProperties的移植
--------------------------------

尽管能力系统可以完成IEEPs(IExtendedEntityProperties)所实现的所有，甚至是更多的功能，这两个概念并不是1:1对应的。在这一节中，我将解释如何将已有的IEEPs转换为能力系统。

下面这个列表将给出IEEP概念的对应能力系统等价：

- Property名称/ID(`String`)：Capability键(`ResourceLocation`)
- 注册(Registration, `EntityConstructing`)：附属(Attaching, `AttachCapabilityEvent<Entity>`)，Capability真正的注册发生在pre-init的时候。
- NBT读写方法：不会自动发生。你需要在事件中附属一个 `ICapabilitySerializable`，并运行 `serializeNBT`/ `deserializeNBT` 来读写NBT数据。

你可能不会需要的特性（如果IEEP只在内部使用）：

- 能力系统提供了一个默认实现概念，用来简化第三方用户的使用，但它对一个内部使用的能力就没那么重要的。如果这个能力只是为了内部使用的话，你可以安全地从Factory返回 `null` 值。
- 能力系统提供了一个 `IStorage` 系统，用来从默认实现中读写数据。如果你选择不提供默认实现，那么 `IStorage` 系统将永远不会被调用，所以它可以留空。

在使用下面这几个步骤之前请先阅读本教程的其它部分，并理解能力系统的概念。

转换指南：

1. 转换IEEP的key/id字符串到 `ResourceLocation`（并使用你的Mod ID作为域）
2. 在你的处理器(Handler)类中（不是你实现能力接口的那个类）创建一个用来储存 `Capability` 实例的字段
3. 将 `EntityConstructing` 事件改为 `AttachCapabilityEvent`，并且你需要依附一个 `ICapabilityProvider`（可能是 `ICapabilitySerializable`，让你能够读写NBT），而不是查询IEEP。
4. 如果你没有注册方法的话，创建一个（你可能已经有一个用来注册IEEP事件处理器(Event Handler)的了），并在其中运行能力的注册函数。

!!! note "译注"

    这篇文档翻译可能有点混乱，如果有看不懂的建议去看[原文](http://mcforge.readthedocs.org/en/latest/datastorage/capabilities/)（虽然原文也不怎么清楚）。

    另外，建议去看Forge这个测试的mod：[TestCapabilityMod.java](https://github.com/MinecraftForge/MinecraftForge/blob/master/src/test/java/net/minecraftforge/test/TestCapabilityMod.java)

    还有capabilities包里的内容：[capabilities包](https://github.com/MinecraftForge/MinecraftForge/tree/master/src/main/java/net/minecraftforge/common/capabilities)

	ustc_zzzz也写了一篇关于Capability很棒的教程，也可以来[这里](https://fmltutor.ustc-zzzz.net/3.3.1-Capability%E7%B3%BB%E7%BB%9F%E4%B8%8E%E5%B7%B2%E6%9C%89%E5%AE%9E%E4%BD%93%E9%99%84%E5%8A%A0%E5%B1%9E%E6%80%A7.html)看一下。