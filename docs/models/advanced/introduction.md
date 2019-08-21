高级模型介绍
========================

尽管简单的模型和方块状态很好，但他们不是动态的。例如，Forge的通用桶可以装各种mod添加的液体。它有动态的模型，基于基础桶模型和液体。它是怎么做的呢？让我们进入`IModel`。

为了理解这是如何工作的，让我们看看内部的模型系统。在本节中，您可能需要参考此内容来清楚地了解正在发生的事情。反过来也是如此。 你可能不会理解这里发生的一切，但是当你浏览这一部分时，你应该能够掌握越来越多的内容，直到一切都清楚。

!!! important "重要"

    如果你是第一次通读，请不要跳过_任何东西_！**为了**有全面的理解，你_必须_读所有的东西！同样，如果这是您第一次阅读，请不用在意页面上的链接，因为那是更深层次的东西。

1. 通过`ModelLoader`加载用`ModelResourceLocation`标记的模型集合

    * 对于物品，他们的模型必须手动标记由`ModelLoader.registerItemVariants`加载(用`ModelLoader.setCustomModelResourceLocation`)完成。

    * 对于方块，它由状态映射得到一个`Map<IBlockState, ModelResourceLocation>`。迭代所有的方块，然后映射中的所有值会被加载。

2. [`IModel`][IModel]由每个`ModelResourceLocation`加载，并缓存至一个`Map<ModelResourceLocation, IModel>`。

    * `IModel`仅由[`ICustomModelLoader`][ICustomModelLoader]加载得到。(多个加载器尝试加载一个模型会报`LoaderException`)。如果没有找到模型，并且`ResourceLocation`实际上是一个`ModelResourceLocation`(也就是，它不是一个普通的模型，而是一个方块状态的变体)，那么他会调用方块状态加载器(`VariantLoader`)。否则的话，该模型是一个正常的原版JSON模型，它会用原版的方式加载(`VanillaLoader`)。

    * 一个原版JSON模型(`models/item/*.json` 或 `models/block/*.json`)加载后，它是一个`ModelBlock`(即使它是物品)。这是一个原版的类，与`IModel`无关。为了纠正它，它被包装到一个`VanillaModelWrapper`里，`VanillaModelWrapper`*实现了*`IModel`。

    * 一个原版/Forge的方块变体加载时，先加载它的完整的方块状态JSON。这个JSON解析到一个`ModelBlockDefinition`里，那里面缓存着JSON的路径。然后变体定义的列表由`ModelBlockDefinition`导出到`WeightedRandomModel`。

    * 当加载一个原版JSON物品模型 (`models/item/*.json`)时，模型需要一个带有变体名`inventory`的`ModelResourceLocation`(例如，泥土方块物品的模型是`minecraft:dirt#inventory`)；从而导致模型由`VariantLoader`加载(尽管它是`ModelResourceLocation`)，若`VariantLoader`加载失败了，则再用`VanillaLoader`。
        * 最重要的副作用是，如果`VariantLoader`加载出错，它会尝试用`VanillaLoader`加载。如果这也出错了，它会导致*两个*stacktrace。第一个是`VanillaLoader`的，第二个是`VariantLoader`的。当调试模型错误时，分析正确的stacktrace很重要。

    * 一个`IModel`可由`ResourceLocation`加载或通过调用`ModelLoaderRegistry.getModel`从缓存中检索或做为其中一个异常处理备选方案。

3. 加载的模型的所有材质依赖会被加载，并放入材质集。材质集是一个巨大的材质，它把所有模型材质粘贴在一起。当模型要渲染一个材质时，它会加上额外的UV偏移，来匹配材质集中材质的坐标。

4. 每个模型调用`model.bake(model.getDefaultState(), ...)`来贴图。返回值[`IBakedModel`][IBakedModel]会被缓存进一个`Map<ModelResourceLocation, IBakedModel>`。

5. 此映射(map)然后被存入`ModelManager`。`ModelManager`是一个存在`Minecraft::modelManager`里的单例，它是私有的，没有getter。

    * `ModelManager`可以不通过反射或访问转换获得，通过`Minecraft.getMinecraft().getRenderItem().getItemModelMesher().getModelManager()` 或 `Minecraft.getMinecraft().getBlockRenderDispatcher().getBlockModelShapes().getModelManager()`。与它们的名字相反，它们是一样的。

    * 可以用`ModelManager`的`ModelManager::getModel`从缓存中获取一个`IBakedModel`(没有加载或/贴图的模型，仅获取已有的缓存)

6. 最后，`IBakedModel`会渲染。这是由`IBakedModel::getQuads`完成的。它的返回值是`BakedQuad`的列表(quadrilaterals: 四边形)。然后它们可以传递给GPU渲染。物品和方块在这里有所不同，但它逻辑相对简单。

    * 原版物品有属性和重载。为了实现它，`IBakedModel`定义了`getOverrides`，它返回一个`ItemOverrideList`。`ItemOverrideList`定义了`handleItemState`，它里面有原始的模型、实体、世界和栈，来找到最终的模型。重载在模型其它所有操作之前完成，包括`getQuads`。因为`IBlockState`不适用于物品，当渲染物品时，`IBakedModel::getQuads`的state参数接受`null`值。

    * 方块有方块状态，当一个方块的`IBakedModel`被渲染时，`IBlockState`直接传入`getQuads`方法。仅在模型的上下文中，方块状态可以有额外的属性，参考[unlisted properties][extended states]。

[IModel]: imodel.md
[IBakedModel]: ibakedmodel.md
[ICustomModelLoader]: icustommodelloader.md
[extended states]: extended-blockstates.md
