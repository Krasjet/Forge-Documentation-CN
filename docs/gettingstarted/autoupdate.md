Forge更新检查器
=============

Forge提供了一个可选的轻量级更新检查框架。它所做的只有检查mod的更新，如果任何mod有可用的更新，在主菜单的Mods按钮上和mod列表中将会显示一个闪光的图标，并会显示对应的更新日志。它**并不会**自动下载更新。

入门
------

你需要做的第一件事情是在 `@Mod` 注解中设定 `updateJSON` 的参数。它的值为一个有效的URL地址指向一个更新JSON文件。这个文件可以放在你的网页服务器中、GitHub上或是任何地方，只要保证mod的所有玩家都能访问这个文件即可。

更新JSON的格式
-----------------------

JSON本身的格式很简单，如下所示：

```javascript
{
  "homepage": "<你的Mod主页/下载地址>",
  "<MC版本>": {
    "<Mod版本>": "<这个版本的更新日志>", 
    // 列出对应Minecraft版本的所有Mod版本，以及它们对应的更新日志
    ...
  },
  ...
  "promos": {
    "<MC版本>-latest": "<Mod版本>",
    // 这是填对应Minecraft版本的Mod的最新版本
    "<MC版本>-recommended": "<Mod版本>",
    // 这里填对应Minecraft版本的Mod的最新稳定版本
    ...
  }
}
```

这个JSON文件已经很清楚了，但要注意以下几点：

- `homepage` 下的链接将会在mod有更新的时候显示给用户。
- Forge使用了一个内部的算法来决定一个版本String是否比另一个更“新”。这个算法应该兼容大部分版本命名方式，如果你不确定你的命名方式是否被支持，请参见 `ComparableVersion` 类。我们强烈建议您使用[语义化版本](http://semver.org/)。
- 更新日志String可以使用 `\n` 分行。一些人可能会选择仅包含一个简略的更新日志，并提供一个链接到完整的更新日志。
- 手动输入这些东西可能会很麻烦。你可以设置一下 `build.gradle`，在构建的时候自动更新这个文件，Groovy有原生的JSON解析支持。这个就留给读者当做练习了。
 
你可以参考[Charset](https://gist.githubusercontent.com/Meow-J/fe740e287c2881d3bf2341a62a7ce770/raw/bf829cdefc84344d86d1922e2667778112b845b1/update.json)和[Botania Unofficial](https://gist.githubusercontent.com/Meow-J/1299068c775c2b174632534a18b65fb8/raw/42c578cf2303aa76d8900f5fdc6366122549d2a8/update.json)的例子构建你自己的JSON。
