# 1 背景
最近在看boost库的协程部分，看得一脸懵逼，在此记录一下的对CPP中实现的协程的理解。

# 2 代码示范

首先看一下的boost库官方的协程示例：https://theboostcpplibraries.com/boost.asio-coroutines

```cpp
#include <boost/asio/io_service.hpp>
#include <boost/asio/spawn.hpp>
#include <boost/asio/write.hpp>
#include <boost/asio/buffer.hpp>
#include <boost/asio/ip/tcp.hpp>
#include <list>
#include <string>
#include <ctime>

using namespace boost::asio;
using namespace boost::asio::ip;

io_service ioservice;
tcp::endpoint tcp_endpoint{tcp::v4(), 2014};
tcp::acceptor tcp_acceptor{ioservice, tcp_endpoint};
std::list<tcp::socket> tcp_sockets;

void do_write(tcp::socket &tcp_socket, yield_context yield)
{
  std::time_t now = std::time(nullptr);
  std::string data = std::ctime(&now);
  
  //向该socket连接的对象的写入数据，如果写入缓冲区中的字节很大，会反复写几次，并发生等待，产生阻塞现象
  //此处使用协程，运行到此处被"挂起"，相当于sleep了，并交出CPU时间片
  //同时，包装该类的"协程"对象（就像线程也有线程对象一样）会保存该协程执行时的上下文
  //至此，在用户态保存了协程的上下文，且记住了挂起时执行到的语句
  async_write(tcp_socket, buffer(data), yield);
  
  //关心的事件发生后，此处便是 IO操作写入完成  这个事件，"挂起"的协程被唤醒
  //开始使用保存的上下文，并根据挂起前的状态，继续执行
  tcp_socket.shutdown(tcp::socket::shutdown_send);
}

void do_accept(yield_context yield)
{
  for (int i = 0; i < 2; ++i)
  {
    tcp_sockets.emplace_back(ioservice);
    
    //和上面注释的同理，在此处挂起，等待异步事件发生后再继续执行
    tcp_acceptor.async_accept(tcp_sockets.back(), yield);
    
    //异步事件发生，在此处被唤醒
    //在这个ioservice中注册一个"协程"类型的事件，等待之后在时间循环中触发，毕竟spawn就是专门给协程用的方法
    //epoll本身支持的事件类型比较有限
    //ioservice是io_context的别名，也就是IO上下文
    spawn(ioservice, [](yield_context yield)
      { do_write(tcp_sockets.back(), yield); });
  }
}

int main()
{
  //这个listen是个同步方法，表示开始监听端口
  tcp_acceptor.listen();
  
  //在ioservice这个上下文实例中注册协程事件
  //需要互相协作的协程才建议注册在一个上下文实例中，要不然会有无意义的让出CPU分片
  spawn(ioservice, do_accept);
  
  //遍历事件列表
  ioservice.run();
}
```

1. 一组协程一定要在一个线程内进行切换的，因为，只有在一个线程内，协程间切换才是不涉及线程间上下文切换这种耗性能的操作。
2. 线程切换时，需要在寄存器和主存之间交换数据来  保存、恢复 线程的现场
3. 线程和协程的挂起没啥区别，但是，上下文的切换有本质的区别：线程切换涉及线程的调度，依赖与线程对应的内核线程对象，会发生用户态和内核态的切换；一组在一个线程内的协程的上下文切换就全都发生用户态中
4. io_service用法：

（1）注册事件和与之对应的回调，能注册的事件就那么几种，但必然是文件的读写，比如，socket_async_read、socket_async_write

（2）注册协程，回调方法就是唤醒另一个准备就绪的协程

（3）run()方法，遍历循环事件，并分发给的先应的回调函数，（同时将分该事件和对应的回调删除，想要继续关心该事件就要继续向io_service注册一下），直到注册的队列为空（也就是用户已经没有关心的事件了）

5. spawn调用时就走后面的lambda表达式的函数体了

上面的协程其实是基于boost::coroutine2的push和pull 非对称协程，同时boost还有另外的对称协程的低层级实现boost::context::callcc，解析来分别介绍这两中协程实现方式

# 3 boost::context::callcc



# 4 boost::coroutine2


