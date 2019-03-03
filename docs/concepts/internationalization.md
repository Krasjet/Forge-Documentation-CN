国际化和本地化
=====================================

国际化, 简称为i18n,是一种可以适用于各种语言的代码设计思路。本地化是指显示适用于用户语言的文字的过程 。

I18n用 _翻译密钥(translation keys)_实现。 翻译密钥是一个字符串，用于标识一段没有特定语言的可显示文本。 例如, 翻译密钥`tile.dirt.name`指的是泥土方块的名字。 这样，可以引用可显示的文本而不关心特定语言。 加入新语言时，代码不需要修改。

本地化将在游戏的本地设置中发生。 在Minecraft客户端中，位置由语言设置指定。 在专用服务器上，唯一受支持的语言环境是en_US。 可以在 [Minecraft Wiki](https://minecraft.gamepedia.com/Language#Available_languages)上找到可用语言环境的列表。

语言文件
--------------

语言文件放在 `assets/[namespace]/lang/[locale].lang`下 (例如 `examplemod`的英语翻译应放在 `assets/examplemod/lang/en_us.lang`下). 资源包格式 3 (在pack.mcmeta中指定)需要小写的区域名。该文件格式只是由`=`分隔的键值对的行组成的。 以`#`开头的行将被忽略。 没有分隔符的行将被忽略。 该文件必须以UTF-8编码。

```properties
# items
item.examplemod.example_item.name=Example Item Name

# blocks
tile.examplemod.example_block.name=Example Block Name

# commands
commands.examplemod.examplecommand.usage=/example <value>
```

若文件中任何地方包含`#PARSE_ESCAPES`注释，会启用更复杂的Java配置文件格式。这种格式支持多行字符串。

!!! note
    启用`＃PARSE_ESCAPES`文件很多地方将会不同。 最重要的是，`:`字符将被视为键值分隔符。 键值对中要使用它必须用`\:`进行转义。

Blocks 和 Items 的用法
---------------------------

!!! note
    2018年7月14日之后，“unlocalized name”一词已被MCP中的“翻译密钥(translation key)”取代。

Block, Item 和一些 Minecraft 的其它类已经内置的显示其名字的翻译密钥。翻译密钥可以通过调用`setTranslationKey(String)`来指定或者重写 `getTranslationKey()`方法。 Item也可以重写`getTranslationKey(ItemStack)`方法来实现不同的 damage 和 NBT有不同的翻译密钥。

默认`getTranslationKey()`会返回 `tile.` 或 `item.` 前提是设置了`setTranslationKey(String)`, ItemBlock会继承它Block的翻译密钥. 另外, `.name` 会把翻译密钥加在它的域之后。例如`item.setTranslationKey("examplemod.example_item")`需要在语言文件中这样设置:

```properties
item.examplemod.example_item.name=Example Item Name
```

不像注册表名,翻译密钥没有命名空间。因此，为了避免冲突，强烈建议在翻译密钥前加上modid。 (例如 `examplemod.example_item`) 。 否则，一旦冲突，一个对象的本地化名会覆盖另一个。

!!! note
    翻译密钥的唯一目的是国际化。 不要将它们用于逻辑。 若要用于逻辑，请改用注册表名称。

    常见的方法是用 `getUnlocalizedName().substring(5)`来分配注册表名. 这并不好,它很脆弱. 请先考虑设置注册表名称，然后使用 `MODID + "." + getRegistryName().getResourcePath()` 来设置翻译密钥.

本地化方法
--------------------

!!! warning
    一个常见问题是让服务器本地化为客户端。 服务器只能在其自己的区域设置中进行本地化，该区域设置不一定与已连接客户端的区域设置相匹配。  

    要考虑客户端的语言设置，服务器应该让客户端使用`TextComponentTranslation`或其他保留语言中翻译密钥的方法在自己的语言环境中本地化文本。

### `net.minecraft.client.resources.I18n` (仅客户端)

**该类仅用于客户端!** 它仅供在客户端上运行的代码使用。 在服务器上使用它会引发异常并崩溃。

- `format(String, Object...)`用格式化在客户端的语言环境中进行本地化. 第一个参数是翻译密钥，其余参数是`String.format(String，Object ...)`的格式化参数。

### `net.minecraft.util.text.translation.I18n` (已废弃)

**该类已被废弃，应避免使用。** 它是用于在逻辑服务器上运行的代码。 由于本地化很少发生在服务器上，因此应考虑其他替代方案。

- `translateToLocal(String)`在没有格式化的情况下,在游戏的语言环境中进行本地化。 该参数是翻译密钥。
- `translateToLocalFormatted(String, Object...)` 用格式化在服务端的语言环境中进行本地化. 第一个参数是翻译密钥，其余参数是`String.format(String，Object ...)`的格式化参数。

### `TextComponentTranslation`

`TextComponentTranslation`是一个`ITextComponent`，它是进行本地化和格式化的简便方法。 在向玩家发送消息时它非常有用，因为它会自动本地化在他们自己的语言环境中。

`TextComponentTranslation(String, Object...)`构造函数的第一个参数是翻译密钥，其余参数用于格式化。 唯一支持的格式说明符是`%1$s`, `%2$s`, `%3$s`等。格式化参数可以是另一个`ITextComponent`，它们的所有属性将被插入到格式化文本的结果中。

### `TextComponentHelper`

- `createComponentTranslation(ICommandSender, String, Object...)`用接收器构造一个`ITextComponent` 区域和本地化设置. 如果接收者是原本客户端，则立刻进行本地化和格式化。 若不是，会用`TextComponentTranslation`完成本地化和格式化。这仅在服务器应允许原本客户端连接时才有用。
