资源
====

资源(Resource)是游戏使用的附加数据，它被储存在数据文件，而不是代码里。资源被储存在类路径(Classpath)的 `assets` 目录中。在默认的 Mod 开发包中，它位于工程的 `src/main/resources` 目录。它包含了模型、纹理、本地化文件等资源。

当多个资源包被应用时，它们会合并。通常状况下，在栈顶的资源包会覆盖它下面的资源包。然而，对于某些文件，比如说本地化文件，多个资源包中数据的内容将会合并。Mod本身其实也定义了一个资源包，在它的 `resources` 目录中，但它们会被认为是“默认”资源包的子集。Mod资源包不能被禁用，但它们可以被其他的资源包所覆盖。

所有的资源都应该使用 snake_case 路径和文件名（所有字母小写，使用“_”作为词语的分隔符），这一项规则在1.11以上是强制的。

`ResourceLocation`
------------------

Minecraft使用 `ResourceLocation` 来识别资源。`ResourceLocation` 包含有两个部分：一个域和一个路径。它通常会指向 `assets/<domain>/<ctx>/<path>` 中的资源，其中 `ctx` 指的是上下文相关的路径部分，它决定于 `ResourceLocation` 是在何处使用的。当 `ResourceLocation` 从一个字符串写入或读取时，它会是 `<domain>:<path>` 这样的形式。如果没有使用域的名字和冒号，当这个字符串由 `ResourceLocation` 解析出来时域几乎都会默认为 `"minecraft"`。Mod需要将它的资源放到与其 modid 相同的域中（比如一个 id 为 `examplemod` 的Mod需要将它的资源放在 `assets/examplemod` 中，并且指向这些文件的 `ResourceLocation` 将会是 `examplemod:<path>`）。这并不是一个必须的要求，并且在某些情况下使用一个不同的（或者多于一个的）域名可能会更好。`ResourceLocation` 在资源系统之外也有应用，它用于识别唯一的对象是一个很棒的方式（比如[注册表][registries])。

[registries]: registries.md