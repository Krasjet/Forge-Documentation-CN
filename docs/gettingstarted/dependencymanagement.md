依赖管理
=====================

Forge对管理和加载mod依赖项有一些支持。 库，甚至其他mod都可以嵌入到构建中，使Forge能够以兼容的方式在运行时提取和加载它们。

mod存储库
------------------

mod存储库是一个类似Maven的存储库，包含mods和库。 此存储库中的工件由其Maven坐标标识：`groupId：groupId:artifactId:version:classifier@extention`。 分类和扩展是可选的。 Forge可以存档，管理和加载此存储库中的mod和库。 mod存储库可能包含多个版本的mod和库，包括快照版本。

如果定义了`Maven-Artifact`清单属性，Forge可以在存储库中存档jar。 此属性的值应为其Maven坐标。

mod存储库支持快照工件。 如果工件版本以`-SNAPSHOT`结尾，则工件将被解析为具有最新时间戳的版本。 时间戳可以设置成清单中`Timestamp`属性，该属性应该是自纪元以来的时间（以毫秒为单位）。


依赖扩展
---------------------

Forge提供了一种在mod中嵌入依赖项和运行时提取它们的简单方法。 通过将依赖jar包放在您自己的jar包中，Forge可以将从mod存储库提取到并加载它们。 这可以用作shading的替代方法，并具有解决依赖项版本冲突的潜在好处。

jar包的包含依赖项由`ContainedDeps`清单属性标记。 它的值应该是一个空格分隔的列表，其中包含将要提取的jar包的名称。 这些jar包应该放在`/META-INF/libraries/{entry}`中。

Forge将检查清单中所包含的jar包，以确定其Maven坐标，以便它可以存档。 如果存在文件`/ META-INF / libraries / {entry} .meta`，Forge将把它读作jar包的清单。 依赖项将根据其`Maven-Artifact`清单属性存储到本地存储库中。

