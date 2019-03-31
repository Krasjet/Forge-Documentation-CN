权限API
=============

权限API是权限系统的一个基础的实现。

它默认的实现不是增加一个先进的权限管理(例如我们知道的PEX)，但它有3个权限级别,(ALL = 所有玩家, OP = 操作者, NONE = 未知是普通玩家还是操作者)。

这些行为可以由mod实现自己的`PermissionHandler`改变

如何使用权限API
-----------------------------

要基础的支持只需要调用 `PermissionAPI.hasPermission(EntityPlayer player, String  node)`，但这默认总是返回false，因为默认的实现使用权限级别`NONE`。所以我们要想要全部玩家，或只有OP才能使用，要注册自己的权限节点。这和检查权限一样简单`PermissionAPI.registerNode(String node, DefaultPermissionLevel level, String description)`，尽管它会在之后完成。

!!! note "提示"

    权限API不限于用于命令，你可以用它干其它事情，如限制对GUI的访问。
    另外，你将它与命令结合，你需要检查`ICommandSender`是否为玩家！

`DefaultPermissionLevel`
--------------

`DefaultPermissionLevel` 有3个值:

* ALL =所有玩家都有此权限
* OP = 只有操作者有此权限
* NONE = 不管是普通玩家还是操作者都有此权限

权限节点
---------------------------------------

虽然技术上没有权限节点的规则，但最佳做法是用`modid.subgroup.permission_id`形式。
建议使用此命名方案，因为其他实现可能具有更严格的规则。

自己实现 `PermissionHandler`
--------------------------------------

默认`PermissionHandler `是非常基础的，但对绝大多数用户已经足够了。但你想在一个大型服务器里控制更多权限。可以通过自定义`PermissionHandler`来实现。

它如何工作以及它的功能完全有你决定，例如你可以做一个简单的实现，只需为每个玩家保存一个文件。
或者你可以使它像PEX一样先进，有数据库支持和许多其他功能。

!!! note "提示"

    并非每个想要使用PermissionAPI的mod都应该更改PermissionHandler，因为同时只能有1个！

首先，您如何实现自己的PermissionHandler完全取决于你，你可以使用文件，数据库或其它任何方式。你所要做的只是实现接口`IPermissionHandler`。在这之后，你需要这样注册它:`PermissionAPI.setPermissionHandler(IPermissionHandler handler)`

!!! note "提示"

    你必须在PreInit阶段设置Handle!
    建议你检查它是否已被其他mod替换。