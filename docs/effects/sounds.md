音效(Sounds)
===========

!!! note

	这个指南写作时Forge最新构建版本为1907，对应Minecraft 1.9。本文应该对任何有新Forge注册系统的版本都是有效的。

目标
----

- 介绍一个Forge中简单实现音效的方式
- Minecraft中有很多版本的 `playSound`。介绍它们的作用。

术语
----


| 术语 | 描述 |
|-----|-----|
| 音效事件(Sound Events) | 触发音效效果的东西，比如说 `"minecraft:block.anvil.hit"` 或 `"botania:spreaderFire"`。 |
| 音效类别(Sound Category) | 音效的类别，比如说 `"player"`, `"block"`  或者是 `"master"`。声音设置GUI里面的滑动条代表了这些类别。 |
| 声音文件(Sound File) | 硬盘上被播放的文件，通常是一个 .ogg 文件。 |

Sounds.json
-----------

这个JSON应该被储存在你的asset目录中

```
src/main/resources/assets/[mod id]/sounds.json
```

这个文件将会告诉原版资源系统你声明了什么音效事件(Sound Events)和对应的音效文件(Sound Files)。

完整的格式可以在原版的[wiki]上找到，我们这里只强调重要的部分，下面是一个例子：

```Json
{
  "openChest": {
    "category": "block",
    "subtitle": "mymod.subtitle.openChest",
    "sounds": [ "mymod:openChestSoundFile" ]
  },
  "epicMusic": {
    "category": "record",
    "sounds": [
      {
        "name": "mymod:music/epicMusic",
        "stream": true
      }
    ]
  }
}
```

在顶级对象之下，我们定义的每一个key都对应一个我们想要告知游戏的音效事件：`"mymod:openChest"` 和 `"mymod:epicMusic"`。注意他们在sounds.json中没有写modid。

在事件之下我们设定音效事件的类别，之后是一个本地化的key来告知有听力障碍的用户音效事件的产生。

最后，我们指定需要播放的音效文件。注意这个值是一个数组——如果我们指定多个音效文件，那么在音效事件触发的时候游戏将会随机从这几个文件中选一个播放。

上面的两个例子代表了两种不同的指定音效文件的办法。[Wiki]中提供了详细的资料，但通常来说，对于像是BGM和音乐唱片之类的长音效文件，请使用第二个形式，因为 `"stream"` 参数告诉Minecraft不要加载整个音效文件到内存中，而是从硬盘中串流(Stream)。使用第二种形式同样允许你指定音量，音调，和音效文件的权重，你可以看原版的[wiki]来了解更多信息。

不论在那种情况下，音效文件的路径都是从 "sounds" asset目录开始。

所以，`"mymod:openChestSoundFile"` 对应路径 `assets/mymod/sounds/openChestSoundFile.ogg`
并且 `"mymod:music/epicMusic"` 对应路径 `assets/mymod/sounds/music/epicMusic.ogg`。

代码注册
-------

然而，在JSON中简单地指定音效事件并不足够。由于1.9对音效系统的改变，你同样需要在代码中注册音效事件。实现非常简单：

```Java
ResourceLocation location = new ResourceLocation("mymod", "openChest");
SoundEvent event = new SoundEvent(location);
GameRegistry.register(event, location);
```

保存好 `SoundEvent` 这个对象，因为你之后播放音效还会用到它。如果你有一个API，并且想让Addon能够播放你的音效事件，请将这些对象放到API中（不要在API中注册它们，你只需要留一些字段在API中并在初始化阶段的时候赋值）。

播放音效
-------

原版有很多用来播放音效方法，然而我们通常会不清楚到底该用哪个。

!!! note

	下面的信息是通过研究并归类这几个不同方法得来的。它们在Forge 1907都是最新的，如果这些东西过时了请告诉我。

注意每个方法都需求一个 `SoundEvent`，你上面注册的那个。同样注意我说**服务端行为(Server Behavior)**和**客户端行为(Client Behavior)**都指的是对应的**逻辑(Logical)端**。如果你想知道这个的区别，请看[这篇文章](../concepts/sides.md)

### `World`

1. <a name="world-playsound-pbecvp"></a>`playSound(EntityPlayer, BlockPos, SoundEvent, SoundCategory, volume, pitch)`
    - 仅仅转发到了[重载(Overload)(2)](#world-playsound-pxyzecvp)，对每个 `BlockPos` 中的坐标都加了0.5
2. <a name="world-playsound-pxyzecvp"></a>`playSound(EntityPlayer, double x, double y, double z, SoundEvent, SoundCategory, volume, pitch)`
    - **客户端行为**：如果传入的玩家是客户端玩家，播放音效事件到客户端玩家。
    - **服务端行为**：对除了被传入的那个玩家以外的任何玩家播放音效事件。 Player可以被设置成 `null`。
    - **用处**：上面两个行为的互补性说明这两个方法都可以从一些在两个逻辑端同时运行，由玩家发起的音效代码中被调用——逻辑客户端将音效播放给用户，逻辑服务端让其他人都听见音效而不让对原始用户重复播放。<br>
  它们也可以同样用来在客户端任一指定地点播放任何声音，只需要传递 `null` 玩家，让所有人都听见即可。
3. <a name="world-playsound-xyzecvpd"></a>`playSound(double x, double y, double z, SoundEvent, SoundCategory, volume, pitch, distanceDelay)`
     - **客户端行为**：仅仅播放音效事件到客户端世界。如果 `distanceDelay` 为 `true`，那么音效的延迟将会取决于声源距玩家的距离。
     - **服务端行为**：无。
     - **用处**：这个方法只在客户端有效，所以它对那些在自定义数据包(Custom Packet)中发送的音效，或其它只在客户端有效的特效类型音效很有用。打雷的时候用的就是这个方法。

### `WorldClient`

1. <a name="worldclient-playsound-becvpd"></a>`playSound(BlockPos, SoundEvent, SoundCategory, volume, pitch, distanceDelay)`
    - 转发到 `World` 的[重载(3)](#world-playsound-xyzecvpd)，对每个 `BlockPos` 中的坐标都加了0.5。

### `Entity`

1. <a name="entity-playsound-evp"></a>`playSound(SoundEvent, volume, pitch)`
    - 转发到 `World` 的[重载(2)](#world-playsound-pxyzecvp)，player的参数传递为 `null`。
    - **客户端行为**：无。
    - **服务端行为**：对该实体(Entity)位置的所有人都播放音效事件。
    - **用处**：在服务端从任何非玩家实体发出声音。

### `EntityPlayer`

1. <a name="entityplayer-playsound-evp"></a>`playSound(SoundEvent, volume, pitch)`（重写(Override)了 [`Entity`](#entity-playsound-evp) 内的那个方法）
    - 转发到 `World` 的[重载(2)](#world-playsound-pxyzecvp)，player的参数传递为 `this`。
    - **客户端行为**：无。可以看一下在 `EntityPlayerSP` 内的[重写](#entityplayersp-playsound-evp)。
    - **服务端行为**：对**除了**这个玩家以外所有玩家播放音效。
    - **用处**：见 [`EntityPlayerSP`](#entityplayersp-playsound-evp)

### `EntityPlayerSP`

1. <a name="entityplayersp-playsound-evp"></a>`playSound(SoundEvent, volume, pitch)` (重写了 [`EntityPlayer`](#entityplayer-playsound-evp) 内的那个方法)
    - 转发到 `World` 的[重载(2)](#world-playsound-pxyzecvp)，player的参数传递为 `this`。
    - **客户端行为**：播放音效事件。
    - **服务端行为**：该方法仅限客户端。
    - **用处**：就像 `World` 内的方法一样，这玩家类内重写的两个方法可以用在同时运行在两端的代码中。客户端负责播放音效到用户，服务端负责让其他人都听见而不对原始用户重新播放。

[wiki]: http://minecraft.gamepedia.com/Sounds.json