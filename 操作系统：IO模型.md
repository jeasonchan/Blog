# 1 参考文章
操作系统IO操作模式   https://blog.csdn.net/u012474535/article/details/80733118

操作系统之多路复用IO总结   https://blog.csdn.net/mortal5/article/details/80952722?depth_1-utm_source=distribute.pc_relevant_right.none-task-blog-BlogCommendFromBaidu-23&utm_source=distribute.pc_relevant_right.none-task-blog-BlogCommendFromBaidu-23

漫谈五种IO模型（主讲IO多路复用）    https://www.jianshu.com/p/6a6845464770

Java NIO 看这一篇就够了  https://blog.csdn.net/u011381576/article/details/79876754

Java NIO IO和NIO的区别   https://www.cnblogs.com/xiaoxi/p/6576588.html

Java NIO 理解分析和基本使用  https://my.oschina.net/u/4347428/blog/3220262


# 2 关键概念理解
同步：发起一个调用，得到结果才返回。

异步：调用发起后，调用直接返回；调用方主动询问被调用方获取结果，或被调用方通过回调函数。

同步阻塞调用：调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。

同步非阻塞调用：在不能立刻得到结果之前，该调用不会阻塞当前线程。

**同步才有阻塞和非阻塞之分；**

阻塞与非阻塞关乎如何对待事情产生的结果（阻塞：不等到想要的结果我就不走了）

# 3 进程状态转换
就绪状态 -> 运行状态：处于就绪状态的进程被调度后，获得CPU资源（分派CPU时间片），于是进程由就绪状态转换为运行状态。

运行状态 -> 就绪状态：处于运行状态的进程在时间片用完后，不得不让出CPU，从而进程由运行状态转换为就绪状态。此外，在可剥夺的操作系统中，当有更高优先级的进程就 、 绪时，调度程度将正执行的进程转换为就绪状态，让更高优先级的进程执行。

运行状态 -> 阻塞状态：当进程请求某一资源（如外设）的使用和分配或等待某一事件的发生（如I/O操作的完成）时，它就从运行状态转换为阻塞状态。进程以系统调用的形式请求操作系统提供服务，这是一种特殊的、由运行用户态程序调用操作系统内核过程的形式。**IO阻塞会隐式让出时间切片！！！**

阻塞状态 -> 就绪状态：当进程等待的事件到来时，如I/O操作结束或中断结束时，中断处理程序必须把相应进程的状态由阻塞状态转换为就绪状态。

![进程状态图](https://img-blog.csdn.net/20180619141100625?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI0NzQ1MzU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

# 4 操作系统IO模型
## 4.1 同步阻塞模型

## 4.2 同步非阻塞模型

## 4.3 IO多路复用


IO多路复用，相当于复用了一个IO中间件，能够在一个线程中处理多个IO请求，把可读的socket告诉用户进程；而传统的处理多个IO的方式是，new出和IO请求同样数量的线程，再让各个线程以同步、异步的IO模型去处理各自负责的的IO

Java NIO（多路复用IO模型） 其实也是阻塞IO模型，只不过解决了同步IO带来的一个线程处理一个IO的 多线程过多导致的server端压力过大问题。基于IO多路复用这个思想，典型的产品、实践：Nginx、Netty

## 4.4 异步IO
