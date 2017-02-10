加载阶段
=======================

Forge加载mod主要分为三个阶段：预初始化(Pre-Initialization)、初始化(Initialization)、后初始化(Post-Initialization)。通常简写为preInit，init，postInit。根据你mod功能的不同，其他的一些事件可能也很重要。由于这三个阶段在不同时间点发生，在每个阶段内能做的事情都是不同的。

!!! note

	加载阶段的事件只能在你的 `@Mod` 类中使用，并且对应的方法上要加上 `@EventHandler` 注解(Annotation)。

预初始化
---------------------------

在预初始化阶段中，你需要让游戏知道你的mod中添加的所有方块、物品等东西。这个阶段对应的事件是 `FMLPreInitializationEvent`。在preInit中一般你可以：

- 注册方块(Block)和物品(Item)到 `GameRegistry`
- 注册Tile Entity
- 注册实体(Entity)
- 分配矿物词典(Ore Dictionary)名字

初始化
---------------------

在初始化阶段中，你需要注册或者完成一些依赖于物品和方块（在preInit中注册）的东西。这个阶段的事件是 `FMLInitializationEvent`。在init中一般你可以：

- 注册世界生成器(World Generator)
- 注册合成配方(Recipe)
- 注册事件处理器(Event Handler)
- 发送IMC(Intermod Communication，Mod间交互)信息

后初始化
----------------------------

在后初始化阶段中，你的mod通常会进行一些依赖或被依赖于其它mod的动作。这个阶段的事件是 `FMLPostInitializationEvent`。在postInit中一般你可以：

- Mod兼容，或者任何需要其它mod的init阶段完成后进行的东西

其它重要的事件
------------

- `IMCEvent`：处理接收到的IMC信息
- `FMLServerStartingEvent`：注册指令(Command)