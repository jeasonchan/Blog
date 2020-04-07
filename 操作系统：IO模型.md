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

![进程状态图](.\resources\进程状态图)

# 4 操作系统IO模型

操作系统的IO模型，主要用在高级语言的底层实现中，在Java中，IO和NIO所使用的底层IO模型是不同的，但是，在代码上并没有十分大的差别，比如，同步非阻塞IO模型，客户端确实不断轮询socket，查看数据是否已经可读，但这一过程**有时候**并不需要Java的使用者来实现，NIO中确实需要Java使用者手动实现对select socket对象的轮询。

作为开发者，我们应该了解知道各种IO模型的基本原理，来应对流量需求、IO需求越来越大的互联网，从而优化server端程序。



## 4.1 同步阻塞模型

同步阻塞IO模型是最建安的IO模型，用户线程在内核进行IO操作时被阻塞，同时会隐式触发用户线程放弃当前的时间切片。

![同步阻塞IO时序图](.\resources\同步阻塞IO时序图.png)

如图所示，用户线程通过系统调用read发起IO读操作，由用户空间转到内核空间。内核等到数据包到达后，**然后将接收的数据拷贝到用户空间**，完成read操作。

用户线程使用同步阻塞IO模型的伪代码描述为：

    {
        read(socket, buffer);
        process(buffer);
    }

即用户需要等待read将socket（这里使用了进程间的通信方式：套接字）中的数据读取到buffer后，才继续处理接收的数据。整个IO请求的过程中，用户线程是被阻塞的，这导致用户在发起IO请求时，不能做任何事情，对CPU的资源利用率不够。


## 4.2 同步非阻塞模型

同步非阻塞IO是在同步阻塞IO的基础上，**将socket设置为NONBLOCK**。这样做用户线程可以在发起IO请求后可以立即返回。

![同步非阻塞IO时序图](.\resources\同步非阻塞IO时序图.png)

如图所示，由于socket是非阻塞的方式，因此用户线程发起IO请求时立即返回。但并未读取到任何数据，并且，**用户线程需要不断主动地发起IO请求**，直到数据到达后，才真正读取到数据，继续执行。

用户线程使用同步非阻塞IO模型的伪代码描述为：

    {
        while(read(socket, buffer) != SUCCESS){
    		//sleep个几微妙 或者  什么都不做    
        };
        process(buffer);
    }

即用户需要不断地调用read，尝试读取socket中的数据，直到读取成功后，才继续处理接收的数据。整个IO请求的过程中，虽然用户线程每次发起IO请求后可以立即返回，但是为了等到数据，**仍需要不断地轮询、重复请求，消耗了大量的CPU的资源**。一般很少直接使用这种模型，而是在其他IO模型中使用非阻塞IO这一特性。

## 4.3 IO多路复用

IO多路复用模型是建立在**内核提供的多路分离函数select**基础之上的，使用select函数可以避免同步非阻塞IO模型中轮询等待的问题，但是！！！用户还是需要轮询select中注册的socket，多个IO请求共享了同一个select轮询。

后来，为了弥补select的不足，先后出现了新的内核函数poll和epoll。

![IO多路复用时序图](.\resources\IO多路复用时序图.png)

如图所示，用户首先将需要进行IO操作的socket添加到select中，然后阻塞等待select系统调用返回。当数据到达时，socket被激活，select函数返回。用户线程正式发起read请求，读取数据并继续执行。
从流程上来看，使用select函数进行IO请求和同步阻塞模型没有太大的区别，甚至还多了添加监视socket，以及调用select函数的额外操作，效率更差。但是，使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。**用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在同一个线程内同时处理多个IO请求的目的。**而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的。

用户线程使用select函数的伪代码描述为：

    {
        select(socket);//注册感兴趣的socket
        while(1) {
            sockets = select();//获取激活的socket
            for(socket in sockets) {
                if(can_read(socket)) {
                read(socket, buffer);
                process(buffer);
                }
            }
        }
    }

其中while循环前将socket添加到select监视中，然后在while内一直调用select获取被激活的socket，一旦socket可读，便调用read函数将socket中的数据读取出来。

然而，使用select函数的优点并不仅限于此。虽然上述方式允许单线程内处理多个IO请求，**但是每个IO请求的过程还是阻塞的（在select函数上阻塞），平均时间甚至比同步阻塞IO模型还要长。**如果用户线程只注册自己感兴趣的socket或者IO请求，然后去做自己的事情，等到数据到来时再进行处理，则可以提高CPU的利用率。


IO多路复用，相当于复用了一个IO中间件，能够在一个线程中处理多个IO请求，把可读的socket告诉用户进程；而传统的处理多个IO的方式是，new出和IO请求同样数量的线程，再让各个线程以同步、异步的IO模型去处理各自负责的的IO。

Java NIO（多路复用IO模型） 其实也是阻塞IO模型，只不过解决了同步IO带来的一个线程处理一个IO的 多线程过多导致的server端压力过大问题。基于IO多路复用这个思想，典型的产品、实践：Nginx、Netty

## 4.4 异步IO

“真正”的异步IO**需要操作系统更强的支持，比如，提供异步请求IO函数**。在IO多路复用模型中，事件循环将文件句柄的状态事件通知给用户线程，由用户线程自行读取数据、处理数据。而在异步IO模型中，内核在IO完成后通知用户线程使用，当用户线程收到通知时，数据已经被内核读取完毕，并放在了用户线程指定的缓冲区内。
![异步IO时序图](.\resources\异步IO时序图.png)

如图所示，异步IO模型中，用户线程直接使用内核提供的异步IO API发起read请求，且发起后立即返回，继续执行用户线程代码。不过此时**用户线程已经将调用的AsynchronousOperation和CompletionHandler注册到内核**，然后操作系统开启独立的内核线程去处理IO操作。当read请求的数据到达时，由内核负责读取socket中的数据，并写入用户指定的缓冲区中。最后内核将read的数据和用户线程注册的CompletionHandler分发给内部Proactor，Proactor将IO完成的信息通知给用户线程（一般通过调用用户线程注册的CompletionHandler），完成异步IO。

用户线程使用异步IO模型的伪代码描述为：

    //随异步请求，一起交付给内核的后处理方法
    void UserCompletionHandler::handle_event(buffer) {
    	process(buffer);
    }
    
    //正式发起异步IO请求
    {
    	aio_read(socket, new UserCompletionHandler);
    }

用户需要重写CompletionHandler的handle_event函数进行处理数据的工作，参数buffer表示Proactor已经准备好的数据，用户线程直接调用内核提供的异步IO API，并将重写的CompletionHandler注册即可。

相比于IO多路复用模型，异步IO并不十分常用，**不少高性能并发服务程序使用IO多路复用模型+多线程任务处理的架构基本可以满足需求。**况且目前操作系统对异步IO的支持并非特别完善，更多的是采用IO多路复用模型模拟异步IO的方式（IO事件触发时不直接通知用户线程，而是将数据读写完毕后放到用户指定的缓冲区中，用户线程通过select查询到数据可读时，进行后续的数据处理操作）。

## 4.5 对比总结

![四种IO模型对比总结](.\resources\四种IO模型对比总结.png)

伪异步IO就是同步非阻塞IO模型，需要用户线程自己轮询内核是否已经处理完毕IO请求。也就是，将数据拷贝到用户空间。