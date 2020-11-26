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

实例：
```cpp
void fibonacci() {
    int result = 0;

    //定义一个函数对象，必须是    continuation&& (*)(continuation&&)  这样的输入输出
    //continuation是只支持转移的，所以，同一个函数的某次执行的栈帧，一般只会有一个有效的continuation，会被不停地被转移来转移去
    auto function = [&result](boost::context::continuation &&continuation_of_caller) {
        int a = 1;
        int b = 1;
        result = a;

        //从resume的源码看，可以看出是先进行jumpContext调用时，跳到了别的地方（移动指向的栈帧，再具体一点，就是指caller这个continuation代表的栈帧），
        //调用jumpContext时，还没来得及执行源码源码里的  return   xxxxx.fctx
        //等从别的地方在跳回来这里后（caller的背景数据可能已经变化)，继续执行剩下的  return  xxxx.fctx  操作
        //之所以还要利用caller.resume()的返回值是因为，要获得最新的caller的背景数据
        continuation_of_caller = continuation_of_caller.resume();

        result = b;

        continuation_of_caller = continuation_of_caller.resume();

        while (continuation_of_caller) {
            result = a + b;
            a = b;
            b = result;
            continuation_of_caller = continuation_of_caller.resume();
        }
        return std::move(continuation_of_caller);
    };

    //看callcc的源码，最关键的一句：
    //return continuation{detail::create_context1< Record >(std::forward< StackAlloc >( salloc), std::forward< Fn >( fn) ) }.resume();
    //先构造一个协程函数Fn的context/continuation，然后执行continuation.resume()，接立刻jumpContext到Fn里面去执行了，Fn执行完或者函数栈帧切出来回到这里
    //会返回Fn最新的context/continuation 并 用来给  continuation_of_Fn  进行初始化
    auto continuation_of_Fn = boost::context::callcc(function);

    for (int i = 0; i < 10; ++i) {
        std::cout << "result:" << result << std::endl;
        continuation_of_Fn = continuation_of_Fn.resume();
    }
}
```

resume源码如下：
```cpp
continuation resume() && {
  BOOST_ASSERT( nullptr != fctx_);
  return { detail::jump_fcontext(
    #if defined(BOOST_NO_CXX14_STD_EXCHANGE)
                 detail::exchange( fctx_, nullptr),
    #else
                   std::exchange( fctx_, nullptr),
    #endif
                    nullptr).fctx };
}
```
经过简化就是：
```cpp
continuation resume() && {
    //检查当前continuation是不是最新的，因为，resume每执行一次，就将continuation的fctx设为nullptr
    //如果是nullptr，可以认为
    BOOST_ASSERT( nullptr != fctx_);
    
    return { detail::jump_fcontext(std::exchange( fctx_, nullptr),nullptr).fctx };
}
```

callcc源码：
```cpp
template< typename StackAlloc, typename Fn >
continuation
callcc( std::allocator_arg_t, StackAlloc && salloc, Fn && fn) {
    using Record = detail::record< continuation, StackAlloc, Fn >;
    return continuation{
                detail::create_context1< Record >(
                        std::forward< StackAlloc >( salloc), std::forward< Fn >( fn) ) }.resume();
}
```
所以，continuation.resume()这个方法其实是分两个时间段执行的：
1. 第一段时间，jumpContext调用造成切换函数栈帧，假设跳去target栈帧
2. 第二段时间，从target回到当前函数栈帧后，resume的返回值是target最新的栈帧数据

# 4 boost::coroutine2


