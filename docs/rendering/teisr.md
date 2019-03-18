TileEntityItemStackRenderer
=======================
!!! note
    该功能仅存在于forge版本>= 14.23.2.2638.

TileEntityItemStackRenderer是一个用OpenGL渲染物品的方法. 该系统比原本的TESRItemStack系统要简单, 它需要一个 TileEntity, 并且不需要获得ItemStack.

使用TileEntityItemStackRenderer
--------------------------

TileEntityItemStackRenderer 允许你用 `public void renderByItem(ItemStack itemStackIn)`渲染物品.  
有一个重载会将partialTicks作为参数，但它永远没在原版中调用。

要使用TEISR，物品首先必须其模型的`IBakedModel#isBuiltInRenderer`返回true
一旦返回true，将访问Item的TEISR进行渲染。 如果它没有，它将使用默认的`TileEntityItemStackRenderer.instance`。

Item设置TEISR，请使用`Item#setTileEntityItemStackRenderer`。 每个Item只能提供一个TEISR，而getter是final的，因此mods不会每帧返回新的实例。

就是这样，使用TEISR不需要额外的设置。

如果需要访问TransformType进行渲染，可以储存通过`IBakedModel＃handlePerspective`传递的TransformType，并在渲染过程中使用它。 始终在`TileEntityItemStackRenderer＃renderByItem`之前调用此方法。