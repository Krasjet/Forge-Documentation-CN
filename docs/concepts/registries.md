注册表
=====

注册是一个让游戏了解一个MOD中的对象（物品、方块、声音等）的过程。注册对象是非常重要的，因为如果没有注册的话，游戏将不能知道Mod中的对象，并会展现出大量不可预料的行为（并且可能会崩溃）。

游戏中大部分需要注册的东西都是由Forge注册表(Registry)来处理的。注册表是一个类似于键值映射的对象。它会自动将整数ID分配到值上。Forge采用的是使用 `ResourceLocation` 作为键的注册表来注册对象，而对象则是 `IForgeRegistryEntry` 类型的。这就让 `ResourceLocation` 变成了对象的“注册表名"。对象的注册表名可以通过 `get`/`setRegistryName` 来访问。Setter只可以被调用一次，如果调用第二次的话会导致异常。每一种可注册对象都有它自己的注册表，在两个不同注册表中的名字将不会冲突。（比如说 `Block` 有一个注册表， `Item` 也有一个注册表，而有着相同名字 `mod:example` 的一个 `Block` 和一个 `Item` 注册时将不会冲突。然而，如果是两个方块使用同一个名字注册，则会导致抛出异常。）`IForgeRegistryEntry`默认的实现（`IForgeRegistryEntry.Impl`）也提供了两个 `setRegistryName` 简便的实现：一个的参数只有一个字符串，另一个的参数是两个字符串。需求一个字符串的重载会检测输入是否包含有一个 `:`（即，它检查是否字符串化的 `ResourceLocation` 存在域），如果没有的化，则使用当前的modid作为资源域。拥有两个参数的重载则将使用 `modID` 作为域，`name` 作为路径，构建一个注册表名。

除此之外也有“注册表的注册表”，一个注册其它注册表的注册表。这个注册表使用 `ResourceLocation` 与一个附加 `Class` 键来注册注册表。这让我们可以查找注册特定类的注册表（比如说，可以通过 `GameRegistry.findRegistry` 来在注册表的注册表中查找 `Block.class` 来获取注册方块的注册表。）

注册对象
-------

注册对象推荐的方式是使用 `RegistryEvent`。在 `RegistryEvent.NewRegistry` 中，注册表应该会被创建。之后，`RegistryEvent.Register` 会在每一个注册表注册的时候被触发。由于 `Register` 是一个泛型事件，事件处理器应该将类型参数设置为被注册对象的类型。这个事件将会包含可以注册对象的注册表(`getRegistry`)，对象可以在注册表中使用 `register`(`All`) 注册。下面是一个注册方块的事件处理器例子：

```java
@SubscribeEvent
public void registerBlocks(RegistryEvent.Register<Block> event) {
    event.getRegistry().registerAll(block1, block2, ...);
}
```

`RegistryEvent.Register` 事件触发的顺序是随机的，除了 `Block` **一定**会第一个被触发，并且 `Item` **一定**会在 `Block` 之后，第二个触发。在 `Register<Block>` 事件被触发之后，所有的[`ObjectHolder`][ObjectHolder] 注解将会被刷新，在 `Register<Item>` 被触发之后，它们会被再次刷新。当其它**所有的** `Register` 事件都触发之后，它们会刷新第三次。

!!! important

	这些事件都在preInit**之前**触发。这也就意味着 `@Mod.EventBusSubscriber`（或在不支持 `static` 的比如Scala mod里 `@Mod` 类的构造器中的 `MinecraftForge.EVENT_BUS.register`）应该用来在preInit之前注册事件处理器。

也有另外一种老的方法可以用来注册对象到注册表，使用 `GameRegistry.register`。任何时候有人建议你使用这个方法，你都应该将其替换为对应正确注册表事件的事件处理器。这个方法使用 `IForgeRegistryEntry::getRegistryType` 找到对应于 `IForgeRegistryEntry` 的注册表，再将对象注册到这个注册表中。还有一个方便的重载需求一个 `IForgeRegistryEntry`(`ifre`)和一个 `ResourceLocation`，它等同于 `GameRegistry.register(ifre.setRegistryName(rl))`。

创建注册表
---------

并不是只有Forge才能有注册表。任何Mod都可以创建它们自己的注册表，并且任何Mod都可以注册东西到其它Mod的注册表中。注册表可以通过 `RegistryBuilder` 来创建。这个类需要一些参数来描述它将生成的注册表，比如说名字，值的 `Class`，以及一些回调函数用于提示注册表的改动。在调用 `RegistryBuilder::create` 的时候，这个注册表就创建了，并且注册至注册表的注册表，并返回到调用者处。

注入注册表值到字段中
------------------

Forge可以直接将注册表中的值注入(Inject)到类的 `public static final` 字段中。这可以通过使用 `@ObjectHolder` 注解类或字段来实现。如果一个类有这个注解，那么它其中所有的 `public static final` 字段都将被作为对象的容器(Object Holder)，并且注解的值为容器的域（即每一个字段都将其作为注入对象注册表名的默认域）。如果一个字段有这个注解，并且值没有域，那么域将会从封装其的类中的 `@ObjectHolder` 注解中获取。如果类也没有包含域，那么这个字段将会被忽略，并会有警告。如果它包含有域的话，那么注入到这个字段的对象就是对应于那个名字的对象。如果类有注解，但其中一个 `public static final` 字段没有的话，对象名字的资源路径将会作为字段的名字。注册表的类型将会为字段的类型。

!!! note

	如果无法找到一个对象，可能是由于对象本身没有被注册，也可能是由于注册表不存在，在此情况下会留下一个调试信息，字段将不会有变动。

由于这些规则有一些复杂，所以下面给出一些例子：

```java
@ObjectHolder("minecraft") // 资源域 "minecraft"
class AnnotatedHolder {
    public static final Block diamond_block = null; // public static final 是必须的
                                                    // 类型为 Block 意味着 Block 的注册表将会被调用
                                                    // diamond_block 是字段名称，由于字段没有被注解，它会作为资源路径
                                                    // 由于没有显式的域，"minecraft" 从类中继承下来
                                                    // 注入的对象：Block注册表中的 "minecraft:diamond_block"

    @ObjectHolder("ender_eye")
    public static final Item eye_of_ender = null;   // 类型为 Item 意味着 Item 的注册表将会被调用
                                                    // 由于注解有值为 "ender_eye"，这将会覆盖字段的名称
                                                    // 由于没有显式的域，"minecraft" 从类中继承下来
                                                    // 注入的对象：Item注册表中的 "minecraft:ender_eye"

    @ObjectHolder("neomagicae:coffeinum")
    public static final ManaType coffeinum = null;  // 类型为 ManaType 意味着 ManaType 的注册表将会被调用。显然这是一个Mod中的注册表
                                                    // 由于注解有值为 "neomagicae:coffeinum"，这将会重写字段的名称
                                                    // 域 "neomagicae" 是显式的，这将会覆盖类的默认值 "minecraft"
                                                    // 注入的对象：ManaType注册表中的 "neomagicae:coffeinum"

    public static final Item ENDER_PEARL = null;    // 注意实际的名称是 "minecraft:ender_pearl"，而不是 "minecraft:ENDER_PEARL"
                                                    // 所以，注入将会失败，这个字段必须要小写或者被注解才行
}

class UnannotatedHolder { // 注意这个类没有注解
    @ObjectHolder("minecraft:flame")
    public static final Enchantment flame = null;   // 这个类没有注解意味着没有能够继承的域
                                                    // 字段的注解提供了对象的所有信息
                                                    // 注入的对象：Enchantment注册表中的 "minecraft:flame"

    public static final Biome ice_flat = null;      // 类与字段都没有注解
                                                    // 所以这个字段将会被忽略

    @ObjectHolder("levitation")
    public static final Potion levitation = null;   // 注解中没有资源域，并且也没有通过类注解指定的默认值
                                                    // 所以，注入将会失败，这个字段需要一个域，或者类需要一个注解
}
```

[ObjectHolder]: #_4