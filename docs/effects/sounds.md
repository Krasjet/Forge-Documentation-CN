音效(Sounds)
===========

术语
----


| 术语 | 描述 |
|-----|-----|
| 音效事件 | 触发音效效果的东西，比如说 `"minecraft:block.anvil.hit"` 或 `"botania:spreaderFire"`。 |
| 音效类别 | 音效的类别，比如说 `"player"`, `"block"`  或者是 `"master"`。声音设置GUI里面的滑动条代表了这些类别。 |
| 音效文件 | 硬盘上被播放的文件，通常是一个 .ogg 文件。 |

`sounds.json`
-------------

这个JSON应该被储存在你的asset目录中：`src/main/resources/assets/<modid>/sounds.json`。这个文件声明了音效事件(Sound Events)和对应的音效文件(Sound Files)。

完整的格式可以在原版的[wiki]上找到，但这个例子中只强调重要的部分：

```json
{
  "open_chest": {
    "category": "block",
    "subtitle": "mymod.subtitle.openChest",
    "sounds": [ "mymod:open_chest_sound_file" ]
  },
  "epic_music": {
    "category": "record",
    "sounds": [
      {
        "name": "mymod:music/epic_music",
        "stream": true
      }
    ]
  }
}
```

!!! important

	和资源系统的其它部分一样，所有东西都应该使用snake_case。

在顶级对象之下，每个key都对应一个声音事件。注意它们在 `sounds.json` 文件中没有modid。每个事件都有一个类别(Category)，以及一个本地化key，它将在字幕打开时显示。最后指定的是需要播放的音效文件。注意这个值是一个数组。如果指定了多个音效文件，游戏将会在音效事件触发时随机选择一个进行播放。

上面的两个例子代表了两种不同的指定音效文件的办法。[Wiki]中提供了详细的资料，但通常来说，对于像是BGM和音乐唱片之类的长音效文件，应该使用第二个形式，因为 `"stream"` 参数告诉Minecraft不要加载整个音效文件到内存中，而是从硬盘中串流(Stream)。使用第二种形式同样允许你指定音量，音调，和音效文件的随机权重。

所有情况下，一个域为 `domain`，路径为 `path` 的音效文件的路径为 `assets/<domain>/sounds/<path>.ogg`。所以，`mymod:open_chest_sound_file` 指向的是 `assets/mymod/sounds/open_chest_sound_file.ogg`，而 `mymod:music/epic_music` 指向的是 `assets/mymod/sounds/music/epic_music.ogg`。

创建音效事件
-----------

为了能够真正地播放音效，我们必须要创建一个对应于 `sounds.json` 中条目的 `SoundEvent`。这个 `SoundEvent` 之后需要[注册]。通常情况下，用于创建音效事件的位置应该设置为它的注册表(Registry)名。

创建一个 `SoundEvent`

```Java
ResourceLocation location = new ResourceLocation("mymod", "open_chest");
SoundEvent event = new SoundEvent(location);
```

这个 `SoundEvent` 会作为这个音效的一个引用，并且会被当做参数传递用来播放音效。所以 `SoundEvent` 需要被存在某个地方。如果一个mod有API，那么它应该在API中展现(Expose)它的 `SoundEvent`。

播放音效
-------

原版有很多用来播放音效方法，然而我们通常会不清楚到底该用哪个。

!!! note

	下面的信息是通过研究并归类这几个不同方法得来的。它们在Forge 1907都是最新的，如果这些东西过时了请告诉我。

注意每个方法都需求一个 `SoundEvent`，也就是你上面注册的那个。而且，**服务端行为(Server Behavior)**和**客户端行为(Client Behavior)**这两个术语都指的是对应的[**逻辑**(Logical)端][sides]。

### `World`

1. <a name="world-playsound-pbecvp"></a> `playSound(EntityPlayer, BlockPos, SoundEvent, SoundCategory, volume, pitch)`
    - 仅仅转发到了[重载(Overload)(2)](#world-playsound-pxyzecvp)，对每个 `BlockPos` 中的坐标都加了0.5
2. <a name="world-playsound-pxyzecvp"></a> `playSound(EntityPlayer, double x, double y, double z, SoundEvent, SoundCategory, volume, pitch)`
    - **客户端行为**：如果传入的玩家是客户端玩家，播放音效事件到客户端玩家。
    - **服务端行为**：对除了被传入的那个玩家以外的任何玩家播放音效事件。 Player可以被设置成 `null`。
    - **用处**：上面两个行为的互补性说明这两个方法都可以从一些在两个逻辑端同时运行，由玩家发起的音效代码中被调用——逻辑客户端将音效播放给用户，逻辑服务端让其他人都听见音效而不让对原始用户重复播放。<br>
  它们也可以同样用来在客户端任一指定地点播放任何声音，只需要传递 `null` 玩家，让所有人都听见即可。
3. <a name="world-playsound-xyzecvpd"></a> `playSound(double x, double y, double z, SoundEvent, SoundCategory, volume, pitch, distanceDelay)`
     - **客户端行为**：仅仅播放音效事件到客户端世界。如果 `distanceDelay` 为 `true`，那么音效的延迟将会取决于声源距玩家的距离。
     - **服务端行为**：无。
     - **用处**：这个方法只在客户端有效，所以它对那些在自定义数据包(Custom Packet)中发送的音效，或其它只在客户端有效的特效类型音效很有用。打雷的时候用的就是这个方法。

### `WorldClient`

1. <a name="worldclient-playsound-becvpd"></a> `playSound(BlockPos, SoundEvent, SoundCategory, volume, pitch, distanceDelay)`
    - 转发到 `World` 的[重载(3)](#world-playsound-xyzecvpd)，对每个 `BlockPos` 中的坐标都加了0.5。

### `Entity`

1. <a name="entity-playsound-evp"></a> `playSound(SoundEvent, volume, pitch)`
    - 转发到 `World` 的[重载(2)](#world-playsound-pxyzecvp)，player的参数传递为 `null`。
    - **客户端行为**：无。
    - **服务端行为**：对处于该实体位置的所有人都播放音效事件。
    - **用处**：在服务端从任何非玩家实体发出声音。

### `EntityPlayer`

1. <a name="entityplayer-playsound-evp"></a> `playSound(SoundEvent, volume, pitch)`（重写(Override)了 [`Entity`](#entity-playsound-evp) 内的那个方法）
    - 转发到 `World` 的[重载(2)](#world-playsound-pxyzecvp)，player的参数传递为 `this`。
    - **客户端行为**：无。可以看一下在 `EntityPlayerSP` 内的[重写](#entityplayersp-playsound-evp)。
    - **服务端行为**：对**除了**这个玩家以外所有玩家播放音效。
    - **用处**：见 [`EntityPlayerSP`](#entityplayersp-playsound-evp)

### `EntityPlayerSP`

1. <a name="entityplayersp-playsound-evp"></a> `playSound(SoundEvent, volume, pitch)` (重写了 [`EntityPlayer`](#entityplayer-playsound-evp) 内的那个方法)
    - 转发到 `World` 的[重载(2)](#world-playsound-pxyzecvp)，player的参数传递为 `this`。
    - **客户端行为**：播放音效事件。
    - **服务端行为**：该方法仅限客户端。
    - **用处**：就像 `World` 内的方法一样，这玩家类内重写的两个方法可以用在同时运行在两端的代码中。客户端负责播放音效到用户，服务端负责让其他人都听见而不对原始用户重新播放。

[wiki]: http://minecraft.gamepedia.com/Sounds.json
[注册]: ../concepts/registries.md#_2
[sides]: ../concepts/sides.md