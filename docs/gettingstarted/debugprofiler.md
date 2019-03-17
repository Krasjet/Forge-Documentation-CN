调试分析器
==========

Minecraft提供了调试分析器(Debug Profiler)，可用于查找耗时的代码。 特别是像`TileEntities`和刷新`TileEntities`这样的东西，对于mod开发者和服务器服主找到耗时代码非常有用。

使用调试分析器
----------------------

调试分析器用起来很简单. 它用 `/debug start`开始分析, 用 `/debug stop`停止。
重要的是，收集数据的时间越多，结果就越好。
建议至少让它收集一分钟的数据。

!!! note "提示"
    当然，您只能分析可用的代码。要分析的实体和TileEntities必须存在于世界中，以显示在结果中。

在你结束调试后,它会在运行文件夹的`debug`子文件夹下创建一个新文件。
它用时间和日期以`profile-results-yyyy-mm-dd_hh.mi.ss.txt`的格式命名。

阅读分析结果
----------------------------

文件顶部是运行时间（以毫秒为单位）和在那段时间内运行了多少tick。

在它下面, 是类似于下面的代码段的信息:
```js
[00] levels - 96.70%/96.70%
[01] |   World Name - 99.76%/96.47%
[02] |   |   tick - 99.31%/95.81%
[03] |   |   |   entities - 47.72%/45.72%
[04] |   |   |   |   regular - 98.32%/44.95%
[04] |   |   |   |   blockEntities - 0.90%/0.41%
[05] |   |   |   |   |   unspecified - 64.26%/0.26%
[05] |   |   |   |   |   minecraft:furnace - 33.35%/0.14%
[05] |   |   |   |   |   minecraft:chest - 2.39%/0.01%
```
下面解释一下每部分的意义:

| [02]| tick | 99.31% | 95.81% |
| :----------------------- | :---------------------- | :----------- | :----------- |
| section的深度 | Section的名字 | 它占上一级Section的时间百分比。 对于第0层，它是tick所用时间的百分比，而对于第1层，它是其父级Section所用时间的百分比 | 第二个百分比是在整个tick中花了多少时间。 |

分析你自己的代码
----------------

调试分析器基本支持`Entity`和`TileEntity`。 如果您想要分析其他内容，您可能需要手动创建sections，如下所示：

```JAVA
  Profiler#startSection(yourSectionName : String);
  //你想要分析的代码
  Profiler#endSection();
```
你可以从`World`, `MinecraftServer`或 `Minecraft` 对象中获取 `Profiler` 对象。
现在，您只需要在生成文件中找到你的Section名称。