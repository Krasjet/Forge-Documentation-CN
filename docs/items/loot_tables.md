战利品表
=======

战利品表(Loot Table)可以非常容易地以指定的随机分布生成战利品。它在原版中用于生成随机的宝箱战利品以及生物掉落。

原版的[Wiki][wiki]非常详细地描述了战利品表的JSON格式，所以这篇文章将会着重于在Mod中操作战利品表的代码。如果想要完全理解这篇文章，请在阅读本文之前先仔细阅读之前提到的[Wiki页][wiki]。

注册一个Mod战利品表
------------------

如果想要让Minecraft加载你的战利品表，可以调用 `LootTableList.register(new ResourceLocation("modid", "loot_table_name"))`，这将会解析并加载 `/assets/modid/loot_tables/loot_table_name.json`。你可以在preInit、init和postInit三个阶段中任意一个阶段调用这个方法。你也可以将战利品表整理到多个文件夹内。

!!! note

	战利品表中的战利品随机池(Loot Pool)必须要附加一个 `name` 标签从而能够在表中唯一识别出这个随机池。通常情况下是使用这个随机池包含的物品种类来命名的。  
	如果你对多个条目(Entry)使用了同一个 `name` 标签（比如同一个物品但施加不同的函数(Function)，那么你必须要给每一个条目一个 `entryName` 标签，以唯一识别出随机池中的条目。对于那些没有冲突的 `name` 标签，`entryName` 将会自动设置为 `name` 的值。  
	这些附加的要求都是由Forge实施的，用于在加载的时候使用 `LootTableLoadEvent` 辅助战利品表的修改（见下）。

注册自定义对象
-------------

除了原版提供的那些，你也可以自己注册你自己的战利品条件(Condition)、战利品函数以及实体属性(Entity Properties)。

实体属性仅仅用于 `minecraft:entity_properties` 这个战利品条件，它可以用来检测参与掠夺(Looting)的实体（被掠夺的实体或者是杀手(Killer)）是否拥有特定的属性。原版中唯一的属性是 `minecraft:on_fire`。

这三个条目的注册非常相似，只需要分别调用 `LootConditionManager.registerCondition`，`LootFunctionManager.registerFunction()` 或 `EntityPropertyManager.registerProperty()` 就可以了。

这些方法都需要传入一个 `Serializer` 的实例，构建它需要对象的ID（`ResourceLocation`）以及代码中实现这些行为的 `Class` —— `LootCondition`，`LootFunction` 或 `EntityProperty`。

由于你注册了JSON的序列化器(Serializer)以及反序列化器(Deserializer)，你可以添加自定义的字段到条件、函数和属性中。见原版 `net.minecraft.world.storage.loot.{conditions, functions, properties}` 包中的实现。

接下来，为了能够使用你的条件、函数、和条件，你需要将它的注册表名传入 `Serializer` 的构造器。下面是一个战利品条目的例子：

```javascript
{
    "type": "item",
    "name": "mymod:myitem",
    "conditions": [
        {
            "condition": "mymod:mycondition",
            "foo": 1, // 可以添加自定义的参数到反序列化器中
        },
        {
            "condition": "minecraft:entity_properties",
            "entity": "this",
            "properties": {
                "mymod:my_property": { // 右边的结构完全取决于反序列化器
                    "bar": 2
                }
            }
        }
    ],
    "functions": [
        {
            "function": "mymod:myfunction",
            "foobar": 3 // 可以添加自定义的参数到反序列化器中
        }
    ]
}
```

修改原版的战利品
---------------

### 综述

你不仅可以添加你自己的战利品表、条件、函数和实体属性，你也可以在加载的时候修改这些项目。

!!! note

	原版允许用户在世界的存档目录中添加自己的战利品表来覆盖游戏（以及Mod）自己的表。这些都被视为是配置(Config)文件，并且**设计上**不能够被以下的方法所修改。

运行时修改战利品表的入口点是 `LootTableLoadEvent`，它会在每个表加载的时候触发。在这里，你可以通过名称查询以及移除随机池，或者添加一个 `Loot Pool` 的实例。这就是为什么Mod中的战利品随机池必须拥有一个名字。

你可能会在想，我们如何才能修改原版的战利品表呢，他们并没有名字。Forge通过对所有的原版战利品表生成名字来解决这个问题。第一个随机池被命名为 `main`，因为大部分的表只有一个随机池。之后的随机池会根据位置命名：第二个随机池是 `pool1`，第三个是 `pool2`，以此类推。删除一个随机池并不会将名字移到别的随机池中。

在每一个 `LootPool` 中，你可以修改随机池的Roll以及Bonus Roll（战利品表将会调用这个随机池多少次），以及通过名字查询和删除条目，或者添加 `LootEntry` 的名字。

和随机池一样，条目也需要有唯一的名字供获取和移除使用。Forge通过增加一个隐藏的 `entryName` 字段到所有的战利品条目中来解决这个问题。如果条目的 `name` 字段在随机池中是唯一的，那么 `entryName` 将会自动设置为 `name`。如果不是的话，则需要手动在Mod的战利品表条目中添加一个，原版的条目则会自动生成。对于每一次的重复，数字会递增。比如说，如果原版中有三个条目，每个都是 `name: "minecraft:stick"`，那么生成的三个 `entryName` 标签则会是 `minecraft:stick`，`minecraft:stick#0` 和 `minecraft:stick#1`。同样，删除一个条目并不会将名字移到别的条目中。

!!! note

	你必须要在战利品表的 `LootTableLoadEvent` 中完成所有对该战利品表所需的改动，之后的任何改动将会被安全检查禁止，如果安全检查被跳过的话则会造成未定义的行为。

### 添加地牢战利品

接下来是修改原版战利品的一个非常常见的案例：添加地牢的物品生成、

首先需要监听要修改的战利品表的事件：

```Java
@SubscribeEvent
public void lootLoad(LootTableLoadEvent evt) {
    if (evt.getName().toString().equals("minecraft:chests/simple_dungeon")) {
        // 使用 evt.getTable() 来获取战利品表
    }
}
```

在这里，我们添加到的是潜在的生成，但我们不想干涉已有随机池的权重。解决这个问题最灵活也是最简单的方法是添加另一个随机池，其中只有一个战利品条目，引用你自己的战利品表，战利品条目可以递归提取自完全不同的战利品表。

比如说，你的Mod可能会包含一个 `/assets/mymod/loot_tables/inject/simple_dungeon.json`：

```javascript
{
    "pools": [
        {
            "name": "main",
            "rolls": 1,
            "entries": [
                {
                    "type": "item",
                    "name": "minecraft:nether_star",
                    "weight": 40
                },
                {
                    "type": "empty",
                    "weight": 60
                }
            ]
        }
    ]
}
```

!!! note

	你仍然需要通过 `LootTableList.register()` 来注册这个表。

这样子战利品条目和随机池就被创建并添加了，地牢箱子将会有一个新的随机池，60%几率什么都不产生，40%的几率产生一个下界之星。

```Java
LootEntry entry = new LootEntryTable(new ResourceLocation("mymod:inject/simple_dungeon"), <weight>, <quality>, <conditions>, <entryName>); // weight 在这里不重要，因为随机池里只有一个条目。其它的参数按你的喜好来设置。

LootPool pool = new LootPool(new LootEntry[] {entry}, <conditions>, <rolls>, <bonusRolls>, <name>); // 其它的参数按你的喜好来设置。

evt.getTable().addPool(pool);
```

当然，如果你添加的战利品不能在这之前决定，你也可以利用类似上面的调用自己构造并添加 `LootPool` 和 `LootEntry` 的实现到事件处理器中。

这种方法实际的例子可以在Botania中找到。事件管理器能在[这里](https://github.com/Vazkii/Botania/blob/e38556d265fcf43273c99ea1299a35400bf0c405/src/main/java/vazkii/botania/common/core/loot/LootHandler.java)找到，注入的战利品表在[这里](https://github.com/Vazkii/Botania/tree/e38556d265fcf43273c99ea1299a35400bf0c405/src/main/resources/assets/botania/loot_tables/inject)。

改变生物掉落
-----------

`EntityLiving` 的子类自动支持死亡时从战利品表中提取战利品。你可以复写 `getLootTable`，将返回值改为想要的战利品表的 `ResourceLocation`。它将是这个生物的默认战利品表，通过设置 `deathLootTable` 的值，不论是你的Mod还是别人的Mod中的生物，你都可以对单独一个实体复写它的战利品表。

在代码中生成战利品
----------------

偶尔你也可能会想在自己的代码中从战利品表生成 `ItemStack`。

首先，你需要获取战利品表本身（你需要能获取到一个 `World`）：

```Java
LootTable table = this.world.getLootTableManager().getLootTableFromLocation(new ResourceLocation("mymod:my_table")); // 解析至 /assets/mymod/loot_tables/my_table.json
```

接下来使用提供的 `LootContextBuilder` 创建一个 `LootContext`，它封装了掠夺的上下文(Context)，比如杀手，掠夺者的幸运(Luck)、以及伤害源。

```Java
LootContext ctx = new LootContext.Builder(world)
    .withLuck(...) // 调整幸运值，通常是 EntityPlayer.getLuck()
    .withLootedEntity(...) // 被掠夺的实体
    .withPlayer(...) // 将玩家设置为掠夺者
    .withDamageSource(...) // 打击(Killing Blow)和非玩家杀手
    .build();
```

最后获取一个 `ItemStack` 的集合：

```Java
List<ItemStack> stacks = table.generateLootForPools(world.rand, ctx);
```

或者填充物品栏(Inventory)：

```Java
table.fillInventory(iinventory, world.rand, ctx);
```

!!! note

	目前只能使用 `IInventory`

[wiki]: https://minecraft-zh.gamepedia.com/%E6%88%98%E5%88%A9%E5%93%81%E8%A1%A8