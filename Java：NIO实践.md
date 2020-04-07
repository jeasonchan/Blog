# 1 前言

Java NIO 看这一篇就够了  https://blog.csdn.net/u011381576/article/details/79876754

Java NIO IO和NIO的区别   https://www.cnblogs.com/xiaoxi/p/6576588.html

Java NIO 理解分析和基本使用  https://my.oschina.net/u/4347428/blog/3220262

NIO 时No-Blocking IO的简称，和IO相比，个人目前已经感知到两个明显的点：

* 面向缓存Buffer，IO面向流
* 程序员手动实现IO多路复用模型种的对select的轮询，减小server端的压力

**（实际上可能远不止上面两个点。）**

# 2 