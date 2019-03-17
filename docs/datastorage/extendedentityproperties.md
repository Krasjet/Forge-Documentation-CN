拓展实体属性
===========

拓展实体属性(Extended Entity Properties, EEP)使我们能对实体(Entity)附加数据。

!!! warning "警告"

	这个系统已经弃用，被[能力](capabilities.md)(Capability)系统所代替。

声明与注册
---------

EEP的基础是 `IExtendedEntityProperties` 这个接口(Interface)。这个结构提供了管理拓展数据的一些基本的方法：

- `init`：使实现(Implementation)知道附加实体的信息，和实体加载的世界
- `saveNBTData`：允许实现在save文件中保存数据，以便实体载入回世界之后读取
- `loadNBTData`：允许实现读取之前保存的数据

实现是一个实现(Implement)这个接口的类，这个类的实例将会附加到实体上，以备存储任何需要的数据。

实现的简单骨架会像这样：

```java
public class ExampleEntityProperty implements IExtendedEntityProperties {
  public static final String PROP_NAME = ExampleMod.MODID + "_ExampleEntityData";

  public static void register() {
    MinecraftForge.EVENT_BUS.register(new Handler());
  }

  // IExtendedEntityProperties内的方法实现

  public static class Handler {
    // 事件处理器(Event Handler)
  }
}
```

附加实现到实体上
--------------

附加拓展属性到实体上是通过处理(Handle) `EntityEvent.EntityConstructing` 这个事件(Event)来完成的，如果检测到目标实体，就调用 `Entity#registerExtendedProperties` 这个方法。

为了能识别你的属性并且避免重复，这个方法需要填写一个String参数作为属性的识别码(Identifier)。一个很好的做法就是将你的modid包括在这个String中，以防与其他mod中的属性冲突。

!!! warning "警告"

	如果同一个属性识别码被添加了两次，Forge将会在识别码上添加一个数字，并通过 `registerExtendedProperties` 方法的返回值返回修改后的识别码。如果你不想这种事情发生，你可以使用 `Entity#getExtendedProperties` 来检查是否这个名字的IEEP被添加过了。

为了处理这个事件，你需要添加如下类似的代码：

```java
@SubscribeEvent
public void entityConstruct(EntityEvent.EntityConstructing e) {
  if (e.entity instanceof EntityPlayer) {
    if (e.entity.getExtendedProperties(PROP_NAME) == null) {
      e.entity.registerExtendedProperties(PROP_NAME, new ExampleEntityProperty());
    }
  }
}
```

使用实现
-------

为了使用拓展数据，你需要先从实体上获取IEEP实现的一个实例，由于实体可能会被卸载或者切换维度(Dimension)，缓冲实例的引用并不安全。

为了获取IEEP引用，我们需要使用 `Entity#getExtendedProperties`，并附上相应的属性ID。返回值如果不是 `null` 的话就是实体构造时对应的 `IExtendedEntityProperties`了。

在你的IEEP实现中创建一个静态 `get` 方法是一个很好的主意，它将自动获取实例，并将其到你的实现类中：

```java
public static ExampleEntityProperty get(Entity p) {
  return (ExampleEntityProperty) p.getExtendedProperties(PROP_NAME);
}
```

保存和读取数据
-------------

Forge允许所有的附加在实体上的IEEP自己有保存和读取功能。然而，注意 `saveNBTData` 和 `loadNBTData` 方法所提供的NBT Tag是实体的全局Tag，它可能保存了其他IEEP的数据，和一些实体本身的数据。

一些情况下访问这些全局数据可能会很有用，但是大多数情况下，防止与现有数据冲突是很重要的，所以我们优先选择将数据储存在嵌套Tag内，并给他一个唯一的名字（比如说IEEP的识别码）。

你的代码会像这样：

```java
@Override
public void saveNBTData(NBTTagCompound compound) {
  NBTTagCompound propertyData = new NBTTagCompound();

  // 写入数据到propertyData

  compound.setTag(PROP_NAME, propertyData);
}

@Override
public void loadNBTData(NBTTagCompound compound) {
  if(compound.hasKey(PROP_NAME, Constants.NBT.TAG_COMPOUND)) {
    NBTTagCompound propertyData = compound.getCompoundTag(PROP_NAME);

    // 从propertyData中读取数据
  }
}
```

与客户端同步数据
--------------

默认情况下，实体的数据将不会被发送到客户端。为了修复这个问题，Mod需要用数据包(Packet)进行自己的同步。

对于下面三种可选的情况你可能会想发送同步数据包：

1. 当实体生成在世界时，你可能需要与客户端同步初始值
2. 当储存的数据改变时，你可能需要通知一些看到实体的客户端
3. 当一个新的客户端开始看到实体，你可能需要提醒它现有的数据

如果你想要更多网络数据包的实现和信息，请去看[网络](../networking/index.md)部分的教程。

可能的代码：

```java
private void dataChanged() {
  if(!world.isRemote) {
    EntityTracker tracker = ((WorldServer)world).getEntityTracker();
    ExampleEntityPropertySync message = new ExampleEntityPropertySync(this);

    for (EntityPlayer entityPlayer : tracker.getTrackingPlayers(entity)) {
      ExampleMod.channel.sendTo(message, (EntityPlayerMP)entityPlayer);
    }
  }
}

private void entitySpawned() {
  dataChanged();
}

private void playerStartedTracking(EntityPlayer entityPlayer) {
  ExampleMod.channel.sendTo(new ExampleEntityPropertySync(this), (EntityPlayerMP)entityPlayer);
}
```

对应的事件处理器：

```java
@SubscribeEvent
public void entityJoinWorld(EntityJoinWorldEvent e) {
  ExampleEntityProperty data = ExampleEntityProperty.get(e.entity);
  if (data != null)
    data.entitySpawned();
}

@SubscribeEvent
public void playerStartedTracking(PlayerEvent.StartTracking e) {
  ExampleEntityProperty data = ExampleEntityProperty.get(e.target);
  if (data != null)
    data.playerStartedTracking(e.entityPlayer);
}
```

在玩家死亡时保持数据
------------------

默认情况下，实体的数据在死亡的时候就会消失。为了改变这一点，在玩家实体重生被克隆时数据需要被复制下来。

这个可以通过处理 `PlayerEvent.Clone` 这个事件来完成。这个事件中，`wasDead` 这个字段可以用来判别是死亡后的重生还是从末地之路返回到主世界。这个检测很重要，因为如果是从末地返回的话数据仍然是存在的。

```java
@SubscribeEvent
public void onClonePlayer(PlayerEvent.Clone e) {
  if(e.wasDeath) {
    NBTTagCompound compound = new NBTTagCompound();
    ExampleEntityProperty.get(e.original).saveNBTData(compound);
    ExampleEntityProperty.get(e.entityPlayer).loadNBTData(compound);
  }
}
```