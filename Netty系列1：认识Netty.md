# 1 前言
参考文章：

Netty入门教程——认识Netty    https://www.jianshu.com/p/b9f3f6a16911

wangzhiwubigdata/God-Of-BigData

先了解一下Netty的特性，对Netty有大概的认知。

Netty 是一个利用 Java 的高级网络的能力，隐藏其背后的复杂性而提供一个易于使用的 API 的客户端/服务器框架。

Maven依赖是：
```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.6.Final</version>
</dependency>
```

# 2 Netty个Tomcat的区别

Netty和Tomcat最大的区别就在于通信协议。

Tomcat是基于Http协议的，Tomcat实质是一个基于http协议（应用层协议）的web容器。但是，Netty不一样，他能通过编程自定义各种（应用层）协议，因为netty能够通过codec自己来编码/解码字节流，完成类似redis访问的功能，这就是netty和tomcat最大的不同。

有人说netty的性能就一定比tomcat性能高，其实不然，tomcat从6.x开始就支持了nio模式，并且后续还有APR模式————一种通过jni（Java Native Interface）调用apache网络库的模式，相比于旧的bio模式，并发性能得到了很大提高，特别是APR模式（毕竟直接基于Native方法编程），而netty是否比tomcat性能更高，则要取决于netty程序作者的技术实力了。

# 3 Netty为什么受欢迎
1. 并发高
2. 传输快
3. 封装好

## 3.1 并发高
### 3.1.1 同步阻塞的server
server只有一个线程，一个socket client的处理完，下一个socket client才能连上。

### 3.1.2 同步非阻塞的server
使用的还是系统的同步阻塞函数（等爱socket连接上），只不过是上层应用通过线程的方式，将请求交给线程处理，server还可以接受新的socket连接请求。

传统的基于BIO的server端实现，accept是阻塞的，当accept到一个具体的client socket实例后，直接新建一个线程来跟这个client socket通讯，并进行相关处理。如果这个client socket长时间"不说话"或者 后续处理的时候有一些耗时等待操作（比如，IO或者网络请求），专门为这个client socket新建出来的线程，在啥也不干，等待IO或者网络完毕的时候，变相浪费了很多内存和一些CPU，因为：

1. Java里每新建一个线程，就会占用2M的堆内存，5000的连接数就要1w Mb的内存，也就是10Gb，大连接数十分消耗内存……
2. 当前线程进行IO或者网络请求被阻塞时，当前线程会自动被挂起，线程调度器会先去执行其他线程，线程的context切换额外消耗一些CPU


基于BIO的server的基础结构模型如下图所示：

![基于BIO的server的架构模型](https://upload-images.jianshu.io/upload_images/1089449-546a563c9822ce16.png?imageMogr2/auto-orient/strip|imageView2/2/w/548/format/webp)

基于BIO的server端流程图如下：

![基于BIO的server端流程图](https://upload-images.jianshu.io/upload_images/1089449-6377fd47256970ef.png?imageMogr2/auto-orient/strip|imageView2/2/w/584/format/webp)

### 3.1.3 基于NIO的多路复用server
底层使用select、poll、epoll这样的NIO函数，相应上层开的server使用也有些变化。


基于NIO多路复用的的server实现，把所有需要等待的操作都注册进selector（可以自己分出多个selector，分别负责IO、Socket什么的），每个selector的循环线程都只需要处理当前selector里就绪的channel，处理运算其实并不怎么消耗时间。最终，selector的注册队列就成了一个缓冲池，有些方法向这个缓冲池中注册channel，这个缓冲池的循环线程负责处理就绪的channel，整个server成了多级的生产者——消费者结构。

如果一个NIO多路复用的server实现，只用了一个selector监视所有channel状态变化，那么它的结构模型是：

![NIO多路复用的单Selector的server的架构模型](https://upload-images.jianshu.io/upload_images/1089449-9eebe781fba495fd.png?imageMogr2/auto-orient/strip|imageView2/2/w/572/format/webp)

NIO多路复用的server端流程图如下：

![NIO多路复用的server端流程图](https://upload-images.jianshu.io/upload_images/1089449-78814cbb3acc30bd.png?imageMogr2/auto-orient/strip|imageView2/2/w/525/format/webp)

此处再回顾一下几种IO模型：

* 同步IO：
    * BIO 函数：
        * 同步阻塞式IO模型，阻塞整个步骤，如果连接少，他的延迟是最低的，因为一个线程只处理一个连接，适用于少连接且延迟低的场景，比如说数据库连接。处理完一个连接的请求，再处理下一个连接。相当于controller和handler在同一个线程中。

        * 同步非阻塞IO模型，不阻塞数据接收但阻塞业务处理，适用于高并发且处理简单的场景，比如聊天软件。

    * NIO 函数：select、poll、epoll，用来实现 多路复用IO 模型

* 异步IO：依赖系统底层的异步IO接口

* 信号驱动IO：这种IO模型主要用在嵌入式开发，我也不太清楚属于同步还是异步，99.99999%是同步的


## 3.2 传输快
Netty的传输快其实也是依赖了NIO的一个特性——零拷贝。

我们知道，Java的内存有堆内存、栈内存和字符串常量池等等，其中堆内存是占用内存空间最大的一块，也是Java对象存放的地方，一般我们的数据如果需要从IO读取到堆内存，需要经过的步骤是：

以从磁盘读取文件，然后再通过socket返回给客户端为例，看一下文件读取、发送过程：

1. DMA将文件从磁盘拷贝到内核的缓冲区
2. CPU从内核缓冲区将文件拷贝到用户缓冲区（文件可能不需要进一步处理）
3. CPU从用户缓冲区将文件拷贝到内核缓冲区的socket缓冲区
4. DMA将socket缓冲区的文件拷贝拷贝到网络设备中

上面的过程共经历了2次DMA拷贝和2次CPU拷贝，如果文件本身比较大，显然这4次拷贝会十分耗时。


Linux系统除了提供上面的传统的IO拷贝函数，还提供了一些新的内核函数，再内核态层面实现零拷贝。


Netty也使用了零拷贝技术，Netty作为一个用户程序，为了实现/利用 内核的 零拷贝 函数，必然是在用户态层面实现对内核函数的调用，同时，Netty还在用户态实现了对数据结构、数据的操作进行了优化，主要是4点：

* Netty提供了CompositeByteBuf类，它可以将多个ByteBuf合并为一个逻辑上的ByteBuf，避免了各个ByteBuf之间的拷贝。
* 通过wrap操作，我们可以将byte[]数组、ByteBuf、 ByteBuffer 等包装成一个 Netty ByteBuf对象，进而避免了拷贝操作。
* ByteBuf支持slice 操作，因此可以将ByteBuf分解为多个共享同一个存储区域的ByteBuf，避免了内存的拷贝。
* 通过FileRegion包装的FileChannel.tranferTo实现文件传输，可以直接将文件缓冲区的数据发送到目标Channel，避免了传统通过循环write方式导致的内存拷贝问题。


## 3.3 封装好
个人认为体现在以下几点：

* API简单易用（封装的足够高级，对于框架使用者来说用起来很简单）
* 扩展性很强（有默认的实现，又能自定扩展）

多说无益，来对比一下使用Java中的BIO、NIO和Netty来实现的一个简单的Socket server端的代码，功能是"回声"，

### 3.3.1 BIO实现server




### 3.3.2 NIO多路复用实现的server




### 3.3.3 Netty实现server
