# 1 前言
参考文章：
为什么单线程的Redis能够达到百万级的QPS？  https://www.toutiao.com/i6802767041762689544/

QPS：query per second
TPS：transaction per second

主要通过以下三点实现：
1. 高效的底层数据结构
2. IO多路复用模型
3. 事件机制

# 2 高效的数据结构
Redis 支持的几种高效的数据结构 string（字符串）、hash（哈希）、list（列表）、set（集合）、zset（有序集合）

# 3 IO多路复用
操作系统一般有四种IO模型：同步阻塞、同步非阻塞、多路复用、异步，redis底层主要调用了的操作系统IO多路复用的函数，极大节省了线程开支。

假设某一时刻与 Redis 服务器建立了 1 万个长连接，对于阻塞式 IO 的做法就是，对每一条连接都建立一个线程来处理，那么就需要 1万个线程，**同时根据我们的经验对于 IO 密集型的操作我们一般设置，线程数 = 2 * CPU 数量 + 1，对于 CPU 密集型的操作一般设置线程 = CPU 数量 + 1**，当然各种书籍或者网上也有一个详细的计算公式可以算出更加合适准确的线程数量，但是得到的结果往往是一个比较小的值，像阻塞式 IO 这也动则创建成千上万的线程，系统是无法承载这样的负荷的更加弹不上高效的吞吐量和服务了。

而多路复用 IO 模型的做法是，用一个线程将这一万个建立成功的链接陆续的放入 event_poll（这个应该是redis的内部实现），event_poll 会为这一万个长连接注册回调函数，当某一个长连接准备就绪后（建立建立成功、数据读取完成等），就会通过回调函数写入到 event_poll 的就绪队列 rdlist 中(目的就是把channel对象标记为就绪的状态)，这样这个单线程就可以通过读取 rdlist 获取到需要的数据。

流程示意图如下：
![redis IO多路复用示意图](https://p1.pstatp.com/large/pgc-image/a2eada5a14e9463185acf22b4b9ae8ef)

Nginx也是采用IO多路复用原理解决C10K问题。c10k：https://www.jianshu.com/p/ba7fa25d3590

# 4 事件机制

redis 客户端与 redis 服务端建立连接，发送命令，redis 服务器响应命令都是需要通过事件机制来做的，如下图（来自互联网的某处...）

![redis 事件机制示意图](https://p9.pstatp.com/large/pgc-image/131366025f844253a98976124973efc6)


1. 首先 redis 服务器运行，监听套接字的 AE_READABLE 事件处于监听的状态下，此时连接应答处理器工作，
2. 客户端与 redis 服务器发起建立连接，监听套接字产生 AE_READABLE 事件，当 IO 多路复用程序监听到其准备就绪后，将该事件压入队列中，由文件事件分派器获取队列中的事件交于连接应答处理器工作处理，应答客户端建立连接成功，同时将客户端 socket 的 AE_READABLE 事件压入队列由文件事件分派器获取队列中的事件交命令请求处理器关联
3. 客户端发送 set key value 请求，客户端 socket 的 AE_READABLE 事件，当 IO 多路复用程序监听到其准备就绪后，将该事件压入队列中，由文件事件分派器获取队列中的事件交于命令请求处理器关联处理
4. 命令请求处理器关联处理完成后，需要响应客户端操作完成，此时将产生 socket 的 AE_WRITEABLE 事件压入队列，由文件事件分派器获取队列中的事件交于命令恢复处理器处理，返回操作结果，完成后将解除 AE_WRITEABLE 事件与命令恢复处理器的关联

# 5 Reactor模式（IO多路复用的设计模式）
大体上可以说 Redis 的工作模式是，reactor 模式配合一个队列，用一个 serverAccept 线程来处理建立请求的链接，并且通过 IO 多路复用模型，自己监听向内核申请资源的socket的状态（用户线程和内核线程通过socket进行通讯，内核做完资源的事情后会改变socket某些状态），一旦某些 socket 的读写事件准备就绪后就对应的事件压入队列rdlist中，然后 worker 工作，由文件事件分派器从中获取事件交于对应的处理器去执行，当某个事件执行完成后文件事件分派器才会从队列中获取下一个事件进行处理。

可以类比在 Netty 中，我们一般会设置 bossGroup 和 workerGroup 默认情况下 bossGroup 为 1，workerGroup = 2 * cpu 数量，这样可以由多个线程（workerGroup）来处理就绪后的事情，但是其中不能有比较耗时的操作如果有的话需要将其放入线程池中，不然会降低其吐吞量。在 redis 中我们可以看做这二者的值都是 1

# 6 补充：为什么说存储的值不宜过大

比如一个 string key = a，存储了 500MB，首先读取事件压入队列中，内核资源准备完毕后，用户和内核通讯的socket告诉用户申请的资源已经就绪，然后，文件事件分派器从中获取到后，交于命令请求处理器处理，此处就涉及到从磁盘中加载 500MB，比如是普通的 SSD 硬盘，读取速度 200MB/S，那么需要 2.5S 的读取时间，此时其它 同样就绪的socket 都将处于等待过程中，就会导致阻塞了 2.5S，同时又会占用较大的带宽导致吞吐量进一步下降
