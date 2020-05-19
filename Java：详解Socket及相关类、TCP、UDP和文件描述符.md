# 1 前言
参考文章：

文件描述符合socket连接    https://www.cnblogs.com/DengGao/p/file_symbol.html

Stream Socket 和 Datagram socket    https://www.xuebuyuan.com/1170964.html

bind()与connect()——计网中socket的使用    https://www.cnblogs.com/Theo-sblogs/p/11130205.html

TCP连接数和文件描述符用尽   https://blog.csdn.net/tsh123321/article/details/88990825


一直很好奇，sokcet的连接在关闭时，做了什么动作？数据库的连接池（个人觉得，本质就是基于socket实现了数据库的私有协议），为什么threadLocal就能避免相关影响？看了一下java.net.Socket 的源码，很有收获。本文通过Socket类的实现来了解套接字及其部分底层原理。

简单交代一下常识：

1. Socket是client实现类，ServerSocket是server端的实现类
2. Stream Socket 是TCP协议的socket实现类，Datagram socket是UDP协议的socket实现类
3. bind是server端用的，用于绑定/监听系统的端口，connect是client用的用于连接到某server。（其实，无论是client和server，都要和系统的端口bind，bind后才能通过端口发/收消息）

# 2 Socket

# 2.1 构造方法
有很多构造方，挑几个我看的懂得……

## 2.1.1 Socket()
最朴素的无参构造方法，实现代码是：

```java
/**
* The implementation of this Socket.
*/
SocketImpl impl;

/**
* Creates an unconnected socket, with the
* system-default type of SocketImpl.
*
* @since   JDK1.1
* @revised 1.4
*/
public Socket() {
    setImpl();
}

/**
* Sets impl to the system-default type of SocketImpl.
* @since 1.4
*/
void setImpl() {
    if (factory != null) {
        impl = factory.createSocketImpl();
        checkOldImpl();
    } else {
        // No need to do a checkOldImpl() here, we know it's an up to date
        // SocketImpl!
        impl = new SocksSocketImpl();
    }
    if (impl != null)
        impl.setSocket(this);
}
```
可见，Socket使用了代理模式，Socket对外暴露的接口，其实最终都是对  SocketImpl impl;  进行操作。

## 2.1.2 Socket(String host, int port)

```java
    /**
     * Creates a stream socket and connects it to the specified port
     * number on the named host.
     * <p>
     * If the specified host is {@code null} it is the equivalent of
     * specifying the address as
     * {@link java.net.InetAddress#getByName InetAddress.getByName}{@code (null)}.
     * In other words, it is equivalent to specifying an address of the
     * loopback interface. </p>
     * <p>
     * If the application has specified a server socket factory, that
     * factory's {@code createSocketImpl} method is called to create
     * the actual socket implementation. Otherwise a "plain" socket is created.
     * <p>
     * If there is a security manager, its
     * {@code checkConnect} method is called
     * with the host address and {@code port}
     * as its arguments. This could result in a SecurityException.
     *
     * @param      host   the host name, or {@code null} for the loopback address.
     * @param      port   the port number.
     *
     * @exception  UnknownHostException if the IP address of
     * the host could not be determined.
     *
     * @exception  IOException  if an I/O error occurs when creating the socket.
     * @exception  SecurityException  if a security manager exists and its
     *             {@code checkConnect} method doesn't allow the operation.
     * @exception  IllegalArgumentException if the port parameter is outside
     *             the specified range of valid port values, which is between
     *             0 and 65535, inclusive.
     * @see        java.net.Socket#setSocketImplFactory(java.net.SocketImplFactory)
     * @see        java.net.SocketImpl
     * @see        java.net.SocketImplFactory#createSocketImpl()
     * @see        SecurityManager#checkConnect
     */
    public Socket(String host, int port)
        throws UnknownHostException, IOException
    {
        //调用下方的private Socket(SocketAddress address, SocketAddress localAddr,boolean stream)
        this(host != null ? new InetSocketAddress(host, port) :
             new InetSocketAddress(InetAddress.getByName(null), port),
             (SocketAddress) null, true);
    }

    //boolean stream 表示是否是TCP协议的Socket，UDP用专门的 DatagramSocket 类
    private Socket(SocketAddress address, SocketAddress localAddr,
                   boolean stream) throws IOException {
        setImpl();

        // backward compatibility
        if (address == null)
            throw new NullPointerException();

        try {
            createImpl(stream);
            if (localAddr != null)
                bind(localAddr);
            connect(address);
        } catch (IOException | IllegalArgumentException | SecurityException e) {
            try {
                close();
            } catch (IOException ce) {
                e.addSuppressed(ce);
            }
            throw e;
        }
    }

```

上面的**私有构造方法**可以看出几个重要的点：

1. 目标的socket地址信息，是必须完整的的，否则报错。
2. client自己的socker地址信息，localAddr可以为null。当localAddr不为null时，会调用bind方法，将client和localAddr表示的socket地址绑定，实现消息发/收；当localAddr为null时，会自动寻找一个空闲的端口绑定。
3. connect(address)，在构造方法里，直接进行了连接操作


## 2.1.3 Socket(InetAddress address, int port)
和2.1.2的构造方法类似，就是host信息，一个是用字符串表示，一个使用InetAddress对象实例表示


## 2.1.4 Socket(String host, int port, boolean stream)
和2.1.2的构造方法最终调用的私有方法类似，区别就是host信息的表示，一个是用字符串表示，一个使用InetAddress对象实例表示

# 3 Stream Socket 和 Datagram socket


# 4 bind、connect、close底层动作分析


# 5 
