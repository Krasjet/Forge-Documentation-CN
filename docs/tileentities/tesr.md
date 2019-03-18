TileEntity特殊渲染器
=========================

`TileEntitySpecialRenderer`（TESR）是用来渲染不能被一个静态的烘焙模型（Static Baked Model，即JSON、OBJ、B3D等）所表示的方块的。TESR需要方块拥有一个TileEntity。

默认情况下OpenGL（`GlStateManager`）是在TESR中处理渲染的。想了解更多的话请看OpenGL的文档。我们推荐尽可能的情况都是使用FastTESR。

创建一个TESR
-----------

如果想要创建一个TESR的话，创建一个继承自 `TileEntitySpecialRenderer` 的类。它会需求一个泛型(Generic)的参数用来指定这个方块的TileEntity类。这个泛型参数将会使用在TESR的 `renderTileEntityAt` 方法中。

对一个特定的TileEntity只存在有一个TESR。所以，世界中存在于单独实例中的值应该存储在传入TESR的TileEntity中，而不是TESR本身。比如说，有一个整数在每一帧都会递增，如果它储存在TESR中的话，它在每一帧、对世界中每一个该类型的TileEntity都会递增。

### `renderTileEntityAt`

这个方法在每一帧都会调用以渲染TileEntity。

#### 参数

- `tileentity`：这是被渲染TileEntity的实例
- `x`, `y`, `z`：TileEntity需要被渲染的位置
- `partialTicks`: 从上一个整刻(Full Tick)之后经过的时间
- `destroyStage`：方块被破坏的破坏阶段

注册TESR
--------

如果想要注册一个TESR的话，可以调用 `ClientRegistry#bindTileEntitySpecialRenderer`，并传入这个TESR所对应的TileEntity类以及TESR的实例。

`FastTESR`
----------

一个TESR可以通过选择继承 `FastTESR` 而不是 `TileEntitySpecialRenderer` 并在 `TileEntity#hasFastRenderer` 中返回 true 来变为一个 FastTESR。使用FastTESR的话需要实现的是 `renderTileEntityFast` 而不是 `renderTileEntityAt`。

FastTESR相比于传统的TESR性能会更好一点，在可能的情况下都应该使用它。这是因为所有的FastTESR实例都被安排到了一起，并且在每一帧只会发送**一个**合并了所有FastTESR的绘制命令到GPU。这个优点的代价是使直接的OpenGL访问（通过 `GlStateManager` 或 `GLXX`）无效。取而代之的是FastTESR必须将顶点添加到所提供的 `VertexBuffer` 中，它表示所有FastTESR合并的顶点数据。这样就可以渲染 `IBakedModel` 了。例子可以在 Forge的 `AnimationTESR` 中找到。