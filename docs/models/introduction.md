模型概述
===============

模型系统是Minecraft给方块和物品设置形状的方式。通过模型系统，方块和物品可以对应到它们的模型。模型系统的主要目的之一是为了资源包可以不仅仅换材质，还可以换方块/物品的整个模型。事实上，每个mod添加的物品和方块都有一个小型的材质包。

`ResourceLocation`类可以把代码连接到文件中的模型和材质。这个类可以从注册系统中识别出模型和材质，但它们的初衷是为了识别文件，它们还可以作为唯一标识符使用。`ResourceLocation`是一个由两个`String`组成的一个简单对象——命名空间和路径。`ResourceLocation`可以表示为`namespace:path`。若创建`ResourceLocation`没有给出明确的命名空间，命名空间默认是`minecraft`。尽管这样，最好还是包含命名空间。

模型系统中，`ResourceLocation`的命名空间直接代表了 `assets/`下的一个文件夹。通常，命名空间和modid一致(例如，原版Minecraft的命名空间是`minecraft`)。`ResourceLocation`的路径部分代表了命名空间下上下文敏感的文件路径。路径意味着什么，确切的路径在哪，要看在哪使用它。例如，如果要一个模型，路径会理解为在`model`下的路径，但如果要一个材质，路径会理解为在`textures`下的路径。因此，`mod:file`前者的语境下是`assets/mod/models/file`，而后者是`assets/mod/textures/file`。如果有东西需要用`ResourceLocation`描述时，它会确切的定义位置在哪。

与模型系统相关的字符串都应用蛇形命名法(尤其是 `ResourceLocation`)  (如: meaning_all_lowercase_and_underscore_separated_words_like_this). Minecraft 1.11之后强制这样使用。
