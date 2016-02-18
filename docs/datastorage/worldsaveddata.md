World Saved Data
================

World Saved Data系统使你能对世界附加数据，你既可以对某个维度(Dimension)进行附加，也可以对整个世界进行附加。

声明
----

这个系统的核心是 `WorldSavedData` 类。这个类提供了以下几个基本方法以供管理数据：

- `writeToNBT`: 允许实现对世界写入数据
- `readFromNBT`: 允许实现对世界读取之前写入的数据
- `markDirty`: 这个方法并不是要由实现重写(Override)的。它需要在数据改变之后被调用，以提醒Minecraft写入这些改变。如果这个方法没有被调用，现有数据将会被暂时保存，`writeToNBT` 将不会被调用。

这个类需要被一个实现(Implementation)进行重写(Override)，并且这个实现的实例(Instance)将会被附加到 `World` 对象上，以备存储任何需要的数据。

实现的简单骨架会像这样：

```java
public class ExampleWorldSavedData extends WorldSavedData {
  private static final String DATA_NAME = MODID + "_ExampleData";

  // 必须的构造器
  public ExampleWorldSavedData() {
    super(DATA_NAME);
  }
  public ExampleWorldSavedData(String s) {
    super(s);
  }

  // WorldSavedData方法
}
```

注册及使用
---------

`WorldSavedData` 类会加载并/或附加在对应的世界上。一个很好的做法是创建一个静态的get方法来加载数据，如果不存在就新建一个新的实例。

附加数据有两种方法：单维度(Per-dimension)，或全局(Global)。全局数据将会被附加在一个公共的地图上，它将在 `World` 类任何一个实例下都可获取，而单世界的数据将不会跨维度共享。注意客户端和服务端是分离的，它们将得到不同实例的全局数据，所以如果数据在两端都需要的话我们需要进行手动同步。

代码中，这些存储位置通过 `MapStorage` 的两个实例呈现在 `World` 对象上。全局的数据通过 `World#getMapStorage()` 获得，单世界地图通过 `World#getPerWorldStorage()` 获得。

现存数据可以通过 `MapStorage#loadData` 获取，新的数据可以通过 `MapStorage#setData` 附加到世界上。

```java
public static ExampleWorldSavedData get(World world) {
  // IS_GLOBAL常量只是为了解释清楚使用的，你应该将其简化为对应的方法
  MapStorage storage = IS_GLOBAL ? world.getMapStorage() : world.getPerWorldStorage();
  ExampleWorldSavedData instance = (ExampleWorldSavedData) storage.loadData(ExampleWorldSavedData.class, DATA_NAME);

  if (instance == null) {
    instance = new ExampleWorldSavedData();
    storage.setData(DATA_NAME, instance);
  }
  return instance;
}
```