绑定模型到方块和物品
=====================================

方块模型
------------

方块不直接连接到模型，而是方块*状态*映射到`ModelResourceLocation`，它指向模型（“模型”包括方块状态JSON）。 `IBlockState`由`IStateMapper`映射到`ModelResourceLocation`。 默认情况下，所有方块的默认状态映射器的工作原理如下：

1. 获得方块状态的方块的注册名
2. 将所述名称设置为`ModelResourceLocation`的`ResourceLocation`
3. 在方块状态中获取所有属性及其值.
4. 使用`IProperty#getNam`获取每个属性的名称
5. 使用`IProperty<T>#getName(T)`获取每个值的名称
6. 对属性名称仅*按字母顺序*对对进行排序
7. 生成以逗号分隔的键值对字符串(例如`a=b,c=d,e=f`)
8. 将其设置为`ModelResourceLocation`的变体部分。
9. 如果变量字符串为空（即未定义属性），则将变体默认为`normal`

变体字符串是自动转成 __小写__ 的，所以如果状态映射器返回`mod:model#VARIANT`，游戏将查询JSON中的字符串“variant”，而不是“VARIANT”。

### 自定义`IStateMapper`

使用自定义的`IStateMapper`很简单。 获取实例后，可以通过调用`ModelLoader.setCustomStateMapper`来注册它。 `IStateMapper`是按每个方块单独注册的，所以这个方法接收`IStateMapper`和它所处理的方块。其它常见的用例是一个构建器`StateMap.Builder`。

#### `StateMap.Builder`

构建器`StateMap.Builder`可以创建一些最常见的`IStateMapper`。 一旦实例化，可以调用方法来设置其参数，并调用`build`来使用这些参数生成`IStateMapper`。

##### `withName`

`withName`以一个属性作为参数，并将为返回的`ModelResourceLocation`设置“名称”（实际上是路径）。 当生成的`IStateMapper`应用于方块状态时，它用给定属性的值，找到该值的名称，并将其用作资源路径。 看一个例子更清楚：

```java
PropertyDirection PROP_FACING = PropertyDirection.create("facing"); // 创建属性
IStateMapper mapper = new StateMap.Builder().withName(PROP_FACING).build(); // 使用构造器
```

现在,如果调用 `mapper` 找方块状态`examplemod:block1[facing=east]`的 `ModelResourceLocation`, 它会把它映射到 `examplemod:east#normal`. 给定`examplemod:block2[color=red,facing=north]`,它会映射到`examplemod:north#color=red`.

##### `withSuffix`

后缀是一个普通字符串，它被添加到资源路径的末尾。 例如，如果后缀设置为`_suff`，则生成的`IStateMapper`会将方块状态`examplemod:block[facing=east]`映射到`examplemod:block_suff#facing=east`。

##### `ignore`

这导致`IStateMapper`在映射块状态时会简单地忽略给定的属性。 调用两次时，两个列表会合并。 一个例子：

```java
PropertyDirection PROP_OUT = PropertyDirection.create("out");
PropertyDirection PROP_IN = PropertyDirection.create("in");
// 这两个是等价的
IStateMapper together = new StateMap.Builder().ignore(PROP_OUT, PROP_IN).build();
IStateMapper merged = new StateMap.Builder().ignore(PROP_OUT).ignore(PROP_IN).build();
```

当要求`together`或`merged`映射方块状态`examplemod:block1[in=north,out=south]`时，它们将给出`ModelResourceLocation` `examplemod:block1#normal`。 给定`examplemod:block2[in=north,out=south,color=blue]`，它将产生`examplemod:block2#color=blue`。 最后，给定`examplemod:block3[color=white,out=east]`（没有`in`），它将产生`examplemod:block3#color=white`。

物品模型
-----------

不像方块可以不需注册自动有一个`IStateMapper`,物品必须手动注册他们的模型,通过调用`ModelLoader.setCustomModelResourceLocation`.这个方法要一个物品,matedate值,和`ModelResourceLocation`,
此方法需要传入一个物品，元数据值和`ModelResourceLocation`，并注册映射，以便所有带有物品和元数据的ItemStack使用给定的模型的`ModelResourceLocation`。 游戏搜索模型的方式如下：

1. 对于 `ModelResourceLocation` `<namespace>:<path>#<varstr>`
2. 尝试寻找主动加载该模型的自定义模型加载器
   1. 如果成功,用找到的加载器加载该模型,并跳出该指导.
3. 如果失败,尝试从方块状态加载器加载.
4. 如果失败,尝试从原版JSON加载器加载(从`assets/<namespace>/models/item/<path>.json`加载)

JSON `models/item`的物品模型可以参考[概述][overrides].

!!! note "提示"
    `ModelLoader.setCustomModelResourceLocation`也要通过给定的物品和`ModelResourceLocation` 调用 `ModelLoader.registerItemVariants`.这为之后的渲染做准备.

### `ItemMeshDefinition`

`ItemMeshDefinition`是一个函数，它接受`ItemStack`并将它们映射到`ModelResourceLocation`。 它们是按物品注册的，这可以通过`ModelLoader.setCustomMeshDefinition`来完成，它接受一个物品和`ItemMeshDefinition`用于它的`ItemStack`。

!!! important "重要"
    `ModelLoader.setCustomMeshDefinition` **不会**调用 `ModelLoader.registerItemVariants`. 因此, 每个`ModelResourceLocation`都必须传递`ModelLoader.registerItemVariants`方法，`ItemMeshDefinition`可以返回它以使其工作。

### 物品的方块状态JSON

请注意，*物品* 可以使用 *方块*状态JSON。 这可以通过简单地将指向块状态JSON的`ModelResourceLocation`传递给`ModelLoader.setCustomModelResourceLocation`或从`ItemMeshDefinition`返回它来实现。 这样做可以让模型利用子模型和组合变体等优点。 两个主要用例是与方块共享模型的物品（特别是`ItemBlock`）和默认的项层模型（组合变量定义中的`textures`块可用于构建模型的层，其中一个属性设置`layer0`，另一个设置`layer1`等）。

!!! note "提示"

    1.9多部分方块状态不能作为开箱即用的物品模型工作，因为它们需要`IBlockState`来选择模型。

!!! important "重要"

    有一个重要的警告。方块状态JSON只能解析`models/block`下模型的路径; 他们找不到`models/item`下的模型（即使使用`../item`也会导致错误）。 这意味着`minecraft:item/generated`模型（设置项的默认变换）不能在方块状态JSON中使用。 一种解决方法是，使用`minecraft:builtin/generated`模型，并使用方块状态JSON中的`transform`标签设置转换。（继承自`minecraft:block/block`的方块模型已经设置了变换，因此这对他们来说不是必需的。）这是一个例子：
    
    ```json
    "defaults": {
      "model": "builtin/generated",
      "__comment": "让Forge为玩家手中，地面上等物品设置默认旋转和比例",
      "transform": "forge:default-item"
    }
    ```

    

[blockstate JSONs]: blockstates/introduction.md
[overrides]: overrides.md
