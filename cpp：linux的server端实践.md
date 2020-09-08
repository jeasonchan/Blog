# 1 背景

参考文档：

(代码实践)linux服务端与window客户端的epoll程序     https://blog.csdn.net/wscxr57/article/details/9930981


（理论总结）SOCKETS - SERVER & CLIENT - 2020    https://www.bogotobogo.com/cplusplus/sockets_server_client.php


（终极实践：TCP+自己实现http）  https://github.com/dashjay/http_demo

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
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>
#include <iostream>
#include <cstring>

namespace jeason {
    const int MAX_EVENTS = 500;
    const int MAX_CHAR_LENGTH = 10000;
}


int main() {
    int epollFD = -1;
    int listenSocket = 0;
    struct sockaddr_in serverAddress;

    //定义并全初始化为0
    char streamBuffer[jeason::MAX_EVENTS][jeason::MAX_CHAR_LENGTH]{0};

    int nCount = 0;

    struct epoll_event epollEvent, epollEvents[jeason::MAX_EVENTS];

    listenSocket = socket(AF_INET, SOCK_STREAM, 0);

    if (-1 == listenSocket) {
        std::cerr << "socket fail\n" << std::endl;
        return -1;
    }

    std::cout << "listenSocket:" << listenSocket << std::endl;

    serverAddress.sin_family = AF_INET;
    serverAddress.sin_addr.s_addr = INADDR_ANY;
    serverAddress.sin_port = htons(8888);//本质就是short int

    int bReuse = 1;
    if (-1 == setsockopt(listenSocket, SOL_SOCKET, SO_REUSEADDR, (char *) &bReuse, sizeof(bReuse))) {
        std::cerr << "setSockOpt fail\n" << std::endl;
        return -1;
    }

    if (-1 == bind(listenSocket, (struct sockaddr *) &serverAddress, sizeof(serverAddress))) {
        std::cerr << "bind fail\n" << std::endl;
        return -1;
    }


    if (-1 == listen(listenSocket, 3)) {
        std::cerr << "listen fail\n" << std::endl;
        return -1;
    }


    epollFD = epoll_create(jeason::MAX_EVENTS);
    if (-1 == epollFD) {
        std::cerr << "epoll_create fail\n" << std::endl;
        return -1;
    }

    std::cout << "epollFD:" << epollFD << std::endl;

    epollEvent.events = EPOLLIN;
    epollEvent.data.fd = listenSocket;

    if (-1 == epoll_ctl(epollFD, EPOLL_CTL_ADD, listenSocket, &epollEvent)) {
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

        nCount = epoll_wait(epollFD, epollEvents, jeason::MAX_EVENTS, -1);
        if (0 > nCount) {
            std::cerr << "epoll_create fail\n" << std::endl;
            continue;
        }


        for (i = 0; i < nCount; i++) {
            if (epollEvents[i].data.fd == listenSocket) {
                fd = accept(listenSocket, (struct sockaddr *) &peeraddr, reinterpret_cast<socklen_t *>(&addrlen));
                printf("peeraddr %s:%d\n", inet_ntoa(peeraddr.sin_addr), ntohs(peeraddr.sin_port));


                epollEvent.events = EPOLLIN;
                epollEvent.data.fd = fd;
                if (-1 == epoll_ctl(epollFD, EPOLL_CTL_ADD, fd, &epollEvent)) {
                    perror("epoll_ctl listen fail\n");
                    return -1;
                }
            } else {
                rcvnum = recv(epollEvents[i].data.fd, streamBuffer[i], 100, 0);
                if (0 == rcvnum) {
                    close(epollEvents[i].data.fd);
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

