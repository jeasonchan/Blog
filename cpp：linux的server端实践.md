# 1 背景

参考文档：

(代码实践)linux服务端与window客户端的epoll程序     https://blog.csdn.net/wscxr57/article/details/9930981

（终极实践：TCP+自己实现http）  https://github.com/dashjay/http_demo

C語言編程網教程（條理清晰)   http://m.biancheng.net/view/2346.html


# 2 linux服务端
监听链接，处理消息，采用的是C的风格，完全面向过程，根据bogotobobo中的总结：

1. create a socket - Get the file descriptor!
2. bind to an address -What port am I on?
3. listen on a port, and wait for a connection to be established.
4. accept the connection from a client.
5. send/recv - the same way we read and write for a file.
6. shutdown to end read/write.
7. close to releases data.

```cpp
#include <sys/socket.h>
#include <sys/epoll.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#include <iostream>
#include <string>

//使用CPP转接过来的C的头，而不是用原始的.h的头文件  更加纯粹的CPP
#include <cstdio>
#include <cstring>
#include <cerrno>
#include <functional>

namespace jeason {
    const int MAX_EVENTS = 500;
    const int MAX_CHAR_LENGTH = 10000;
}

inline void exit_with_flag_errInfo(std::string &&customFlag) {
    std::cout << "customFlag:" << customFlag << std::endl
              << "err no:" << errno << std::endl
              << "error info:" << strerror(errno) << std::endl;
    exit(errno);
}

int main() {
    //定义并全初始化为0，
    char streamBuffer[jeason::MAX_EVENTS][jeason::MAX_CHAR_LENGTH]{0};
    int nCount = 0;

    //获取socket文件描述符
    int serverSocketFD = -1;
    serverSocketFD = socket(PF_INET, SOCK_STREAM, 0);

    if (-1 == serverSocketFD) {
        std::cerr << "socket fail\n" << std::endl;
        return -1;
    }
    std::cout << "listenSocketFD:" << serverSocketFD << std::endl;


    //在协议层面，修改的socket的属性
    int bReuse = 1;
    if (-1 == setsockopt(serverSocketFD, SOL_SOCKET, SO_REUSEADDR, (char *) &bReuse, sizeof(bReuse))) {
        std::cerr << "setSockOpt fail\n" << std::endl;
        return -1;
    }

    //绑定监听的端口
    sockaddr_in serverAddress{AF_INET, htons(8888), {INADDR_ANY}};


    /*
    以下是C风格的结构体赋值方法，很显然，使用花括号初始化是直接初始化，省去了先初始化再赋值的开销

    serverAddress.sin_family = AF_INET;
    serverAddress.sin_addr.s_addr = INADDR_ANY;
    serverAddress.sin_port = htons(8888);//本质就是short int
    */

    if (-1 == bind(serverSocketFD, (struct sockaddr *) &serverAddress, sizeof(serverAddress))) {
        std::cerr << "bind fail\n" << std::endl;
        return -1;
    }

    //等待client端接入，开始监听
    if (-1 == listen(serverSocketFD, 3)) {
        std::cerr << "listen fail\n" << std::endl;
        return -1;
    }

    //创建epoll队列，队列大小为500
    int epollQueueFD = -1;
    epollQueueFD = epoll_create(jeason::MAX_EVENTS);
    if (-1 == epollQueueFD) {
        std::cerr << "epoll_create fail\n" << std::endl;
        return -1;
    }
    std::cout << "epollQueueFD:" << epollQueueFD << std::endl;

    //设置该epoll队列所关注的事件和文件描述符
    struct epoll_event epollEventConfig, epollEventsQueue[jeason::MAX_EVENTS];
    //设置队列的所关注的事件类型是EPOLLIN
    epollEventConfig.events = EPOLLIN;
    //设置队列的所关注的事件是发生在该FD上的事件
    epollEventConfig.data.fd = serverSocketFD;

    //
    if (-1 == epoll_ctl(epollQueueFD, EPOLL_CTL_ADD, serverSocketFD, &epollEventConfig)) {
        std::cerr << "epoll_create fail\n" << std::endl;
        return -1;
    }


    while (true) {
        int i = 0;
        struct sockaddr_in peeraddr;
        int addrlen = sizeof(peeraddr);
        int fd;
        int rcvnum = 0;
        int r = 9;

        memset(&peeraddr, 0, sizeof(peeraddr));

        nCount = epoll_wait(epollQueueFD, epollEventsQueue, jeason::MAX_EVENTS, -1);
        if (0 > nCount) {
            std::cerr << "epoll_create fail\n" << std::endl;
            continue;
        }

        //对得到的nCount个事件进行遍历
        for (i = 0; i < nCount; i++) {
            if (epollEventsQueue[i].data.fd == serverSocketFD) {
                fd = accept(serverSocketFD, (struct sockaddr *) &peeraddr, reinterpret_cast<socklen_t *>(&addrlen));
                printf("peer addr %s:%d\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port));


                epollEventConfig.events = EPOLLIN;
                epollEventConfig.data.fd = fd;
                if (-1 == epoll_ctl(epollQueueFD, EPOLL_CTL_ADD, fd, &epollEventConfig)) {
                    perror("epoll_ctl listen fail\n");
                    return -1;
                }
            } else {
                rcvnum = recv(epollEventsQueue[i].data.fd, streamBuffer[i], 100, 0);
                if (0 == rcvnum) {
                    close(epollEventsQueue[i].data.fd);
                    continue;
                }
                streamBuffer[i][99] = 0;
                std::cout << "recv:streamBuffer\n" << streamBuffer[i] << std::endl;
                memset(streamBuffer[i], 0, 100);
            }
        }
    }

    return 0;
}
```


# 3 linux客户端
发送消息到server端

