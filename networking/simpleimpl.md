SimpleImpl
==========

SimpleImpl是一个围绕着 `SimpleNetworkWrapper` 类的数据包(Packet)系统. 使用这个系统是至今在客户端与服务端之间发送自定义数据最简单的方法了.

入门
---------------

首先你需要创建你自己的 `SimpleNetworkWrapper` 对象. 我们推荐您把它放到一个单独的类里面，比如说像是 `ModidPacketHandler`. 在这个类里面将你的 `SimpleNetworkWrapper` 创建为一个静态字段(Static Field):

```java
public static final SimpleNetworkWrapper INSTANCE = NetworkRegistry.INSTANCE.newSimpleChannel("mymodid");
```

其中 `mymodid` 是你的数据包管道(Packet Channel)的标识符，一般来说是你的mod ID，除非它非常的长.

制作数据包(Packet)
--------------

### IMessage

一个数据包被定义为使用 `IMessage` 接口(Interface)的类. 这个接口定义了2个方法(Method)， `toBytes` 和 `fromBytes`. 这些方法分别向 `ByteBuf` 的对象中写入(`toBytes`)与读取(`fromBytes`)你数据包的数据，`ByteBuf` 是一个用来保存发送的字节流(数组)的对象.

举个例子，我们定义一个小的数据包，它能够传输一个整数(int)对象:

```java
public class MyMessage implements IMessage {
  // 默认的构造器(Constructor)是必须的
  public MyMessage(){}

  private int toSend;
  public MyMessage(int toSend) {
    this.toSend = toSend;
  }

  @Override public void toBytes(ByteBuf buf) {
    // 写入int到buf对象
    buf.writeInt(toSend);
  }

  @Override public void fromBytes(ByteBuf buf) {
    // 从buf对象里读取int. 注意如果你写入了多个值，读取的时候要按照写入的顺序读取.
    toSend = buf.readInt();
  }
}
```

### IMessageHandler

现在，我们该如何使用这个数据包呢? 首先，我们需要有一个能够**处理(Handle)**这个数据包的类. 这个类需要实现 `IMessageHandler` 接口. 比如说我们想要把我们之前传输的那个整数在服务端当做给一个玩家钻石的数量. 这个处理器(Handler)应该是这样的:

```java
// IMessageHandler的参数是<REQ, REPLY>，也就是说第一个是你接收的包，第二个是你返回的包. 返回的包可以被用作发送包的回应(Response).
public class MyMessageHandler implements IMessageHandler<MyMessage, IMessage> {
  // 注意，默认的构造器是需要的，但是在这里是隐式定义的

  @Override public IMessage onMessage(MyMessage message, MessageContext ctx) {
    // 这是发送到服务器的数据包发送到的玩家
    EntityPlayerMP serverPlayer = ctx.getServerHandler().playerEntity;
    // 发送的值
    int amount = message.toSend;
    serverPlayer.inventory.addItemStackToInventory(new ItemStack(Items.diamond, amount));
    // 没有回应数据包
    return null;
  }
}
```

为了管理的方便，我们建议(但不要求)您将这个类设置为 `MyMessage` 类的内部类(Inner Class). 如果你这样做了，注意这个类也应该被声明为静态类(`static`).

!!! warning

    在Minecraft 1.8中，数据包默认由网络线程(Network Thread)来处理(而不是主线程).

    这意味着你的 `IMessageHandler` **不能**直接操作游戏中的大部分对象. 也就是说上面的例子不再正确. Minecraft提供了一个简单的方法让你的代码执行在主线程上: 

	使用 `IThreadListener.addScheduledTask`.

	你可以使用 `Minecraft` 实例(客户端)或者是 `WorldServer` 实例来获取`IThreadListener`(译注: `Minecraft` 实例(客户端)与 `WorldServer`实例(服务端)都实现了 `IThreadListener`.你可以将他们获取并创建为一个 `mainThread` 对象来使用.我自己把这个MyMessage重新写了一遍正确的，地址在这里:[http://git.io/vqhqF](http://git.io/vqhqF))

注册数据包
-------------------

现在我们有了数据包，有了数据包的处理器. 但是 `SimpleNetworkWrapper` 还需要一个步骤才能工作! 为了让它能够使用数据包，这个数据包必须要注册一个**识别码**，识别码是一个用来在客户端和服务端之间映射数据包类型的一个整数. 想要注册一个数据包，调用:

```java
INSTANCE.registerMessage(MyMessageHandler.class, MyMessage.class, 0, Side.Server);
```

这里面的 `INSTANCE` 使我们之前定义的 `SimpleNetworkWrapper`.

这是一个有些复杂的方法，我们来看一看每一个参数的用处.

- 第一个参数是 `messageHandler`，也就是处理你数据包的类. 这个类必须要有一个默认构造器，并且绑定的REQ类要和第二个参数一样.
- 第二个参数是 `requestMessageType`，它是真正的数据包类. 这个类必须有一个默认构造器，并且需要和前一个参数绑定的REQ类一样.
- 第三个参数是这个数据包的识别码，对单独一个通道(Channel)来说它是唯一的，我们推荐您使用一个静态变量来保存ID，调用 `registerMessage` 时使用 `id++`. 这100%保证了ID是唯一的.
- 第四个也是最后一个参数是是你数据包的<font color=red>**接收端**</font>，如果你想同时向客户端和服务端发送这个数据包，它必须被用<font color=red>**不同的**</font>识别码注册两次.

使用数据包
-------------

当发送数据包的时候，确保**在接收端**注册了该数据包的处理器. 如果没有处理器，数据包会在网络中传输之后被丢弃，变成一个“泄漏”数据包(译注: 即一直占据内存不被释放，直到程序结束. 曾经wiki上一个教程导致很多mod都遭受这个问题). 这虽然这在不必要的网络带宽之外没什么影响，我们建议您还是要修复的.

### 发送到服务端

发送数据包到服务端只有一种方式. 因为只存在**一个**服务端，当然只有**一种**方式发送到服务端. 想要实现这个目标，我们需要再次使用我们之前定义的 `SimpleNetworkWrapper`.调用

```java
INSTANCE.sendToServer(new MyMessage(toSend))
```

这个消息将会发送到注册为 `Side.SERVER` 的 `IMessageHandler`，如果它存在的话.

### 发送到客户端

发送数据包到客户端一共有4种方式.

1. `sendToAll` - 调用 `INSTANCE.sendToAll` 将会发送数据包到服务器上的所有玩家，不管他们在什么地方，在什么维度(Dimension).
2. `sendToDimension` - `INSTANCE.sendToDimension` 有两个参数，一个 `IMessage` 和一个整数. 整数是将要发送的维度的ID，它可以由 `world.provider.dimensionID` 获得. 这个数据包将会被发送到在指定维度所有的玩家.
3. `sendToAllAround` - `INSTANCE.sendToAllAround` 需要 `IMessage` 和一个 `NetworkRegistry.TargetPoint` 对象. 在 `TargetPoint` 之内所有玩家都将收到这个数据包. 创建一个 `TargetPoint`对象需要维度(见第2条), x/y/z坐标，范围. 它代表了在世界里的一个立方体.
4. `sendTo` - 最后，发送到单独的一个客户端可以使用 `INSTANCE.sendTo`. 它需求 `IMessage` 和一个 `EntityPlayerMP`，就是要发送到的玩家. 注意，尽管这并不是更加泛化的 `EntityPlayer`，只要你在服务端你就能转换任何 `EntityPlayer` 到 `EntityPlayerMP`.