# 1 前言

Java NIO 看这一篇就够了  https://blog.csdn.net/u011381576/article/details/79876754

Java NIO IO和NIO的区别   https://www.cnblogs.com/xiaoxi/p/6576588.html

Java NIO 理解分析和基本使用  https://my.oschina.net/u/4347428/blog/3220262

Java进阶（五）Java I/O模型从BIO到NIO和Reactor模式     http://www.jasongj.com/java/nio_reactor/

NIO 时No-Blocking IO的简称，和IO相比，个人目前已经感知到两个明显的点：

* 面向缓存Buffer，IO面向流
* 程序员手动实现IO多路复用模型种的对select的轮询，减小server端的压力

**（实际上可能远不止上面两个点。）**


IO多路复用这种具体方法，一般都是包含在Reactor模式中，用来提高IO性能，同时，高度依赖系统的异步api的高性能IO模式还有Proactor：

Reactor模式     https://blog.csdn.net/xiaocaidexuexibiji/article/details/11135803

高性能IO之Reactor模式     https://www.cnblogs.com/doit8791/p/7461479.html

两种高性能 I/O 设计模式 Reactor 和 Proactor     https://www.cnblogs.com/daoluanxiaozi/p/3274925.html





# 2 
