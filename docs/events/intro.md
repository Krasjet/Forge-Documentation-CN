事件
====

Forge使用的是一种事件总线(Event Bus)的机制，使mod能够从原版或者mod行为中截取事件(Event)。

例子：一个事件可以用来使在原版木棍右键的时候执行一个行为。

对于大部分事件的主事件总线位于 `MinecraftForge.EVENT_BUS`。有一些对于特定事件的（比如说地形生成）其它的总线也在同一个类中。

一个事件处理器(Event Handler)是一个包含有一个或者多个 `public void` 成员方法，并且每一个方法都有 `@SubscribeEvent` 注解的类。

创建一个事件处理器
----------------

```java
public class MyForgeEventHandler {
    @SubscribeEvent
    public void pickupItem(EntityItemPickupEvent event) {
        System.out.println("Item picked up!");
    }
}
```

这个事件处理器监听了 `EntityItemPickupEvent`。正如名字所述，这个事件将会在一个 `Entity` 捡起一个物品的时候发送到事件总线。

要想注册一个事件处理器，请使用 `MinecraftForge.EVENT_BUS.register()`，并将你的事件处理器的一个实例传进去。

!!! note

	在更早的Forge版本中存在有两个分开的事件总线。一个是Forge的，一个是FML的。然而这个系统已经被弃用很长时间了，所以你以后不需要再使用FML的事件总线了。

### 静态事件处理器

事件处理器也可以是静态的。用于处理的方法仍是用 `@SubscribeEvent` 来注解，和实例处理器(Instance Handler)唯一的不同只是这个方法使用 `static` 标记了。要想注册一个静态事件处理器，你不能传入类的实例，你必须要将 `Class` 本身传递进去。一个例子：

```java
public class MyStaticForgeEventHandler {
    @SubscribeEvent
    public static void arrowNocked(ArrowNockEvent event) {
        System.out.println("Arrow nocked!");
    }
}
```

它必须要使用 `MinecraftForge.EVENT_BUS.register(MyStaticForgeEventHandler.class)` 来注册。

### 自动注册静态事件处理器

一个类也可以使用 `@Mod.EventBusSubscriber` 来进行注解。当 `@Mod` 类构造时，这样的类将会自动注册至 `MinecraftForge.EVENT_BUS`。这完全等同于在 `@Mod` 类的构造器最后加上 `MinecraftForge.EVENT_BUS.register(AnnotatedClass.class);`

!!! note

	使用这个注解并不会注册一个类的实例。它注册的是类本身（即事件处理方法必须是静态的）。

取消
----

如果一个事件可以被取消(Candel)，它将会被注解为 `@Cancelable`，并且 `Event#isCancelable()` 方法将会返回 `true`。一个可取消事件的取消状态可以通过调用 `Event#setCanceled(boolean canceled)` 来变更，在这个函数内传入 `true` 将取消这个事件，而传入 `false` 的话则重新启用这个事件。然而，如果这个事件不能被取消（通过 `Event#isCancelable()` 来定义），不论传入什么boolean值都将抛出 `UnsupportedOperationException`，因为不可取消的事件中的取消状态是不可变的。

!!! important

	不是所有的事件都可以被取消！尝试取消一个不可被取消的事件将会导致抛出 `UnsupportedOperationException`，这很可能会导致游戏崩溃！在尝试取消一个事件之前确保先使用 `Event#isCancelable()` 来检测它是否能够被取消。

结果
----

一些事件会有一个 `Event.Result` 结果(Result)。可能的结果有三个：`DENY` 停止事件、`DEFAULT` 使用原版行为、`ALLOW` 强制动作发生，不论原本动作是否要发生。一个事件的结果可以通过调用 `setResult`，并传入一个 `Event.Result` 参数。并不是所有的事件都有结果，一个有结果的事件将会被注解为 `@HasResult`。

!!! important

	不同的事件可能对结果(Result)有着不同的用法，使用结果之前请先查看事件对应的JavaDoc。

优先级
------

事件处理器方法（被 `@SubscribeEvent`注解的方法）有一个优先级(Priority)。你可以对事件处理器方法的注解加上 `priority` 值作为参数从而设置其优先级。优先级可以是 `EventPriority` 里的任何值(`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`)。有着 `HIGHEST` 优先级的事件处理器将会最先被执行，之后沿着降序执行直到 `LOWEST` 的事件处理器被最后执行。

子事件
------

很多事件都有它们自己不同的变种。一个事件的变种虽然不同但它们都基于共同的因子（比如说 `PlayerEvent`），或者是一个有多个阶段的事件（比如说 `PotionBrewEvent`）。要注意的是，如果你监听了父事件类，这个事件监听器将会在该父事件的**任一**子类触发时被调用。