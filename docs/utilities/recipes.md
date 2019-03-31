合成
=======

随着Minecraft 1.12的更新,Mojang引入了一种基于JSON文件的新型数据驱动的合成系统。在那之后Forge也改成了这个，Minecraft 1.13中拓展成了数据包。

加载配方
---------------
Forge会加载在`./assets/<modid>/recipes/`目录下的所有配方。你可以随意调用这些文件，考虑到原版约定是在输出物品之后命名它们。 此名称也用作注册密钥，但不影响配方的操作。

!!! note "提示"

    配方文件不能以下划线开头，因为这是为静态文件保留的。 JSON文件扩展名是必需的。

配方文件
---------------

一个基本的配方文件应该如下:

```json
{
    "type": "minecraft:crafting_shaped",
    "pattern":
    [
        "xxa",
        "x x",
        "xxx"
    ],
    "key":
    {
        "x":
        {
            "type": "forge:ore_dict",
            "ore": "gemDiamond"
        },
        "a":
        {
            "item": "mymod:myfirstitem",
            "data": 1
        }
    },
    "result":
    {
        "item": "mymod:myitem",
        "count": 9,
        "data": 2
    }
}
```

!!! note "提示"

    当您首次获得原版合成的配方时，它将自动解锁合成书中的配方。 要达到同样的效果，您必须使用[成就(未完成)][Advancements]系统并为每种合成创建新的成就。
    
    如果一个成就存在，并不意味着它可在成就树中被看见。

### 类型 (Type)

合成的类型。你可以把它当作定义用哪一种合成布局，例如 `minecraft:crafting_shaped` (有序合成)或 `minecraft:crafting_shapeless`(无序合成).

你也可以定义你自己的合成类型，只需要使用[`_factories.json`][Factories]文件。

### 组 (Groups)

你还可以把你的配方增加到一个组里，这样可以在合成帮助接口里一起显示。group字符串相同的配方会放在同一个组里。例如，这可以用于所有的门的合成配方，这样即使有很多种不同的门，在合成帮助里只会有一个条目。

配方的类型
-----------------------------
这样的话，我们可以仔细看一下有序合成和无序合成定义上的区别。

### 有序合成

有序合成需要`pattern`和`key`两个关键字。pattern定义物品的排序，它必须以占位符的形式。每一个物品你都可以选择任意的一个字符作为占位符。而key定义了占位符对应的物品。可以添加附加属性`forge:ore_dict` ，它可以定义物品是[矿物词典][OreDictionary]的一部分。例如，无论什么铜矿都可以生成铜锭。要这样的效果的话，要用`org`标签定义物品，而不是用`item`定义那个物品。默认有[很多][Wiki]这样的类型，你也可以自己定义。`data`是一个可选标签，用于定义方块或物品的metadata。

!!! important  "重要"

    用了`setHasSubtypes(true)`的物品必须要`data`字段。如果不用，那么意味着有着任何metadata的该物品都可以用于合成。例如，不定义剑的数据意味着用了一半的剑也可以用于合成!


### 无序合成

无序合成不需要`pattern`和`key`关键字。

要定义一个无序合成，你要使用`ingredients`列表。它定义了合成中要用的物品，也可以用`forge:ore_dict`它函数式的声明在上方。默认有[很多][Wiki]这样的类型，你也可以自己定义。它甚至可以一个对象定义多个实例，意味着合成时必须要在合成台里放多个这样的物品。

!!! note "提示"

    你的配方里有多少ingredients没有限制，原版合成台没有只允许一个合成里放9个物品。

下面这个例子展示了ingredients在JSON里是什么样的:

```json
    "ingredients": [
        {
            "type": "forge:ore_dict",
            "ore": "gemDiamond"
        },
        {
            "item": "minecraft:nether_star"
        }
    ],
```

### 烧炼
要定义一个熔炉的烧炼，你要用 `GameRegistry.addSmelting(input, output, exp);`，因为烧炼现在不是基于JSON的。

合成元素
---------------

### Patterns

pattern用`pattern`列表定义。每一个字符串代表合成网格中的一行，每个占位符代表一列。在上面的例子中可以看到空格表示那个位置不需要物品。

### Keys

Key集合用于绑定pattern，其键为列表里的占位符。一个key可以定义多个物品，例如木制按钮。这意味着玩家可以列表中的任意一个合成，例如不同种的木头。

```json
  "key": {
     "#": [
      {
        "item": "minecraft:planks",
         "data": 0
      },
      {
        "item": "minecraft:planks",
        "data": 1
      }
    ]
  }
```

### Results

每个合成必须有result标签用于定义输出。

有时当合成东西时，你可以得到不止1个物品。这是由定义`count`数值实现的。如果忽略了它，意味着它没有输出方块，它默认为1.不能是负数因为Itemstack不能小于0.除result外，其它地方不能使用`count`。

`data`字段是可选的，用于定义方块或物品的metadata。不存在是默认为0.

!!! note "提示"

    使用`setHasSubtypes(true)`的物品必须要data字段，这种情况下它不是可选的。

<a id="factories">工厂</a>
---------
工厂可以定义配方和自定义类型的原料。要创建一个工厂，先创建一个`_factories.json`，在这个文件中定义合成的类型，例如 `recipes`,  `ingredients` 和 `conditions`这些类型各自代表了`IRecipeFactory `, `IIngredientFactory`, 和`IConditionFactory`，主键是可以之后用于配方的`name，`值是全类名，这个类必须实现上面的一个接口。该类必须有空的构造方法。例如:

```json
{
    "<type>": {
        "<name>": "<fully qualified classname for the specified type>"
    }
}
```

!!! note "提示"

    没必要每种类型都创建一个新的`_factories.json`，它们可以定义在一个文件中。

### 条件配方

条件配方可以通过上述工厂系统创建，为此你可以用`conditions`类型和 上面的`IConditionFactory` 一起用，然后你可以把`conditions`添加到你的配方中:

```json
{
    "conditions": [
        {
            "type": "<modid>:<name>"
        }
    ]
}
```
这些条件仅适用于整个配方，而不是一种原料。例如，你想用已有的条件`forge:mod_loaded`, 和`"modid": "<mod to check>"`检查一个mod是否加载。

!!! note "提示"

    条件只会在启动时检查。

常量
---------
它可以为你的配方定义常量。这些值定义在`_constants.json`中，只需要在你mod的配方里写`#<name>`。对装满的桶，你应该使用 `fluid`而不是`data`。例如，这个常量定义了`#SADDLE为原版的马鞍。

```json
[
	{
		"name": "SADDLE",
		"ingredient": {
			"item": "minecraft:saddle",
			"data": 0
		}
	}
]
```

[Wikipedia]: https://en.wikipedia.org/wiki/Factory_(object-oriented_programming)
[OreDictionary]: ../utilities/oredictionary.md
[Advancements]: #
[Wiki]: https://minecraft.gamepedia.com/Recipe
[Factories]: #factories
