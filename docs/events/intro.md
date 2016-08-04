事件(Events)
===========

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

取消(Cancel)与结果(Result)
-------------------------

如果一个事件可以被取消(Cancel)，它将会被注解为 `@Cancelable`。事件可以通过调用 `setCanceled`，并传入一个boolean值来标示这个事件是否被取消了。如果一个事件不可以被取消，`IllegalArgumentException` 将会被抛出。

!!! important

	不同的事件可能对结果(Result)有着不同的用法，使用结果之前请先查看事件对应的JavaDoc。

一些事件会有一个 `Event.Result`。结果可能有三个：`DENY` 停止事件、`DEFAULT` 使用原版行为、`ALLOW` 强制动作发生，不论原本动作是否要发生。一个事件的结果可以通过调用 `setResult`，并传入一个 `Event.Result` 参数。并不是所有的事件都有结果，一个有结果的事件将会被注解为 `@HasResult`。

优先级(Priority)
---------------

事件处理器方法（被 `@SubscribeEvent`注解的方法）有一个优先级(Priority)。你可以对事件处理器方法的注解加上 `priority` 值作为参数从而设置其优先级。优先级可以是 `EventPriority` 里的任何值(`HIGHEST`、`HIGH`、`NORMAL`、`LOW`、`LOWEST`)。有着 `HIGHEST` 优先级的事件处理器将会最先被执行，之后沿着降序执行直到 `LOWEST` 的事件处理器被最后执行。

子事件(Sub Event)
----------------

很多事件都有它们自己不同的变种。一个事件的变种虽然不同但它们都基于共同的因子（比如说 `PlayerEvent`），或者是一个有多个阶段的事件（比如说 `PotionBrewEvent`）。要注意的是，如果你监听了父事件类，这个事件监听器将会在该父事件的任一子类触发时被调用。