矿物词典
=======

矿物词典(OreDictionary)主要是为了Mod间兼容而存在。

已注册到矿物词典的物品将能够代替其它拥有相同矿物词典名的物品。这样就可以使用以上任一物品合成相同的结果。

虽然名字是“矿物(Ore)”词典，但是它也可以使用在非矿物的物品上。只要一个物品与另一个物品（比如染料）相似，就都可以注册进矿物词典，并通过矿物词典调用。

矿物词典名称规范
---------------

!!! note "提示"

	由于矿物词典名称需要在不同Mod间共享，它们应该是比较统一的。请使用其它Mod可能会使用的名称。

Forge并没有规定名称需要有某种特定的格式，但下面这些规则现在已经是比较流行的标准了：

整个矿物词典名称通常使用驼峰命名法(camelCase，使用小写字母开头，接下来的单词首字母大写的复合词)，并且不要使用空格以及下划线。

矿物词典名称的第一个单词应该指明物品的类型。对于那些特殊的物品（比如`record`，`dirt`，`egg`，`vine`），一个单词就足够了，不用指明物品类型。

名称的最后一部分应当指明物品的材料。这将区分开`ingotIron`与`ingotGold`。

如果两个单词还是不够详细，你也可以加上第三个单词。比如说，花岗岩被注册为`blockGranite`，而磨制花岗岩被注册为`blockGranitePolished`。

如果想要一份通用前缀以及后缀的列表，见[通用矿物词典名称](#_5)。

WILDCARD_VALUE
--------------

这个值表示 `ItemStack` 的metadata不重要。使用范例[见下](#_3)。

在合成配方中使用矿物词典
---------------------

使用矿物词典的配方在创建与注册上与普通的合成配方差别不大。主要的区别就是逆序要使用一个矿物词典名称而不是一个特定的 `Item` 或 `ItemStack`。

如果想要让一个配方使用矿物词典条目，创建一个 `ShapelessOreRecipe` 条目或者 `ShapedOreRecipe` 实例，并且调用 `GameRegistry.addRecipe(IRecipe recipe)` 注册它。

!!! note "提示"

	你可以通过调用 `OreDictionary.doesOreNameExist(String name)`，检测返回值是否为一个有效的 `ItemStack` 来验证矿物词典名称的有效性。

在合成中矿物词典的另一种用途就是 `[WILDCARD_VALUE]()`，你可以将 `OreDictionary.WILDCARD_VALUE` 传入一个 `ItemStack` 的构造器。

!!! note "提示"

	`OreDictionary.WILDCARD_VALUE` 应该只被用在配方的输入中，在配方输出中使用 `WILDCARD_VALUE` 只会改变输出 `ItemStack` 的损害值(Damage)。

注册物品至矿物词典
----------------

你应该在 `FMLPreInitializationEvent` 阶段，在初始化需要注册的方块以及物品之后添加条目到矿物词典。

只需要调用 `OreDictionary.registerOre(ItemStack stack, String name)`，并传入包含物品或方块以及其metadata值，就能够将其注册至矿物词典。

你也可以调用 `OreDictionary.registerOre` 的重载函数，直接传递 `Block` 或 `Item` 即可，不需要自己手动创建一个 `ItemStack` 了。

如果需要一个 `ItemStack` 矿物词典命名的帮助，见[矿物词典名称规范](#_2)。

通用矿物词典名称
--------------

原版Minecraft物品以及方块的矿物词典名称可以在 `net.minecraftforge.oredict.OreDictionary` 中找到。完整的列表将不会包含在这里。

在矿物词典中已经使用的通用前缀包括 `ore`，`ingot`，`nugget`，`dust`，`gem`，`dye`，`block`，`stone`，`crop`，`slab`，`stair`，和 `pane`。

在Mod物品中的通用前缀包括 `gear`，`rod`，`stick`，`plate`，`dustTiny`，和 `cover`。

在矿物词典中已经使用的通用后缀包括 `Wood`，`Glass`，`Iron`，`Gold`，`Leaves`，和 `Brick`。

在Mod物品中的通用前缀包括金属的名称（`Copper`，`Aluminum`，`Lead`，`Steel` 等）以及其它材料。