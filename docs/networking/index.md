网络(Networking)
===============

服务器与客户端之间的交流是一个成功的mod实现的关键。

阅读[概述][overview]部分来了解为什么服务端与客户端通信至关重要，还有一些思考网络部分的基本的技巧。

Forge提供了很多技巧来辅助通信 - 大部分都是基于[Netty]实现的。

对于一个新的mod，最简单的方式就是使用[SimpleImpl]，这个系统使Netty复杂的系统通过抽象层简化了。它使用的是Message与Handler系统。

[Netty]: http://netty.io "Netty主页"
[SimpleImpl]: simpleimpl.md "SimpleImpl详细介绍"
[overview]: overview.md "网络概述"
