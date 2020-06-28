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





# 2 NIO 入门介绍

NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。









## 2.1 传统IO和NIO简单对比

```java
package default_package.NIO练习;

import org.springframework.util.SystemPropertyUtils;

import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;

public class Main {


    /*
    文件的流操作，务必注意的在Finally中进行流关闭，此例暂不考虑这些
     */
    public static void main(String[] args) throws IOException {
        //==========用IO流读取文件===========
        long start = System.currentTimeMillis();

        FileInputStream fileInputStream = new FileInputStream("test.txt");

        BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);

        byte[] bytesBuffer = new byte[1024 * 4];//1个page的大小，符合IO读取的设计逻辑
        int readLength = 0;

        while ((readLength = bufferedInputStream.read(bytesBuffer)) != -1) {
            System.out.println(new String(bytesBuffer, 0, readLength, StandardCharsets.UTF_8));
        }

        //IO流操作，哪一步是从用户态切换到内核态，也就是哪一步会形成IO阻塞？？？？？


        //=================NIO==========================
        fileInputStream = new FileInputStream("test.txt");
        FileChannel fileChannel = fileInputStream.getChannel();
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024 * 4);

        byteBuffer.clear();//以防万一，先clear一下

        while (-1 != (readLength = fileChannel.read(byteBuffer))) {
            byteBuffer.flip();//需要将buffer从读完的状态切换到写入状态时调用该方法

            while (byteBuffer.hasRemaining()) {
                //!!!!!!!!!!!!!!!!!!!!!!!!!
                // byteBuffer.get()  会移动当前的指针索引，就跟Iterator.next()差不多
                // 如果 只 byteBuffer.array()  而不进行索引移动，会一直死循环，因为 byteBuffer.hasRemaining()  始终为true
                System.out.println(new String(byteBuffer.array(), 0, readLength, StandardCharsets.UTF_8));
            }

            byteBuffer.compact();//

        }

        //======================================================


    }


}

```

## 2.2 Buffer介绍

使用Buffer一般遵循下面几个步骤：

- 分配空间（ByteBuffer buf = ByteBuffer.allocate(1024); 还有一种allocateDirector后面再陈述
- 写入数据到Buffe：readLength = fileChannel.read(byteBuffer)，
- 调用filp()方法：byteBuffer.flip()
- 从Buffer中读取数据（System.out.print((char)byteBuffer.get());）
- 调用clear()方法或者compact()方法

Buffer顾名思义：缓冲区，实际上是一个容器，**一个连续数组**。Channel提供从文件、网络读取数据的渠道，但是读写的数据都必须经过Buffer。如下图：

![Buffer和Chann数据传输示意图.png](.\resources\Buffer和Chann数据传输示意图.png)

向Buffer中写数据时可通过两种方式：

- 从Channel写到Buffer (fileChannel.read(buf))
- 通过Buffer的put()方法 （buf.put(…)）

从Buffer中读取数据也有两种方式：

- 从Buffer读取到Channel (channel.write(buf))
- 使用get()方法从Buffer中读取数据 （buf.get()）

### 2.2.1 capacity， position， limit， mark

可以把Buffer**简单地**理解为一组基本数据类型的元素数组，它通过几个索引变量来保存这个数据的当前位置状态：capacity， position， limit， mark：

| 索引     | 说明                                                    |
| -------- | ------------------------------------------------------- |
| capacity | 缓冲区数组的总长度                                      |
| position | 下一个要操作的数据元素的位置                            |
| limit    | 缓冲区数组中不可操作的下一个元素的位置：limit<=capacity |
| mark     | 用于记录当前position的前一个位置或者默认是-1            |

现在通过几个典型状态了解一下，四个索引的具体含义。

**使用ByteBuffer.allocate(10)创建一个Buffer时，四个索引指向的位置示意图**：

![创建Buffer时四个索引指向位置.png](.\resources\创建Buffer时四个索引指向位置.png)

创建一个长度是10的字节数组，capacity的值为10。创建完毕后，position=0，limit和capacity的数值大小都是"长度"，**如果直接作为索引，必然会访问越界！！！！！！！！！！**

```java
package default_package.NIO练习.Buffer用法;

import java.nio.ByteBuffer;

public class Main {
    public static void main(String[] args) {
        ByteBuffer buffer = ByteBuffer.allocate(10);
        buffer.put("1234567890".getBytes());
        System.out.println((char) buffer.get(buffer.capacity() - 1));


        /*
        打印：
        0
         */
    }
}
```



**向缓冲区写入5个字节后，索引的指向的位置发生变化：**

![新Buffer写入数据后四个索引指向位置.png](.\resources\新Buffer写入数据后四个索引指向位置.png)

正如一开始定义的那样，postion指向了“下一个要操作的数据元素的位置”，index为5的位置，也就是第6个Byte为位。其他的索引没有变化。



**要将Buffer写入Channel，也就是从Buffer中取处已经写入的数据：**

在读取数据之前，要先调用ByteBuffer.flip()方法，执行的操作就是：**position设回0，并将limit设成之前的position的值，便于后面按照作左闭右开原则进行读取。**

![Buffer的flip方法效果.png](.\resources\Buffer的flip方法效果.png)

这时底层操作系统就可以从缓冲区中正确读取这个5个字节数据并发送出去了，读取时，**直接按照左闭右开原则，读取数组的[position，limit)索引范围内的值**。在下一次**写数据之前**我们再调用clear()方法，缓冲区的索引位置又回到了Buffer刚创建时的位置。



**调用clear()方法：**

position将被设回0，limit设置成capacity，换句话说，Buffer被清空了，**其实Buffer中的数据并未被清除，**只是这些标记告诉我们可以从哪里开始往Buffer里写数据。如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先写些数据，那么使用compact()方法。



**compact()方法：**

将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。



**Buffer.mark()方法：**

可以标记Buffer中的一个特定的position，之后可以通过调用Buffer.reset()方法**恢复到这个position**。



**Buffer.rewind()方法：**

将position设回0，limit保持不变，仍然表示能从Buffer中读取多少个元素，所以你可以重读Buffer中的所有数据。



## 2.3 SocketChannel

NIO的强大功能部分来自于Channel的非阻塞特性，套接字的某些操作可能会无限期地阻塞。

例如：

对accept()方法的调用可能会因为等待一个客户端连接而阻塞，对于server端而言，这阻塞完全没有必要，甚至浪费资源；

对read()方法的调用可能会因为没有数据可读而阻塞，直到连接的另一端传来新的数据。

总的来说，创建/接收连接或读写数据等I/O调用，都可能无限期地阻塞等待，直到底层的网络实现发生了什么。慢速的，有损耗的网络，或仅仅是简单的网络故障都可能导致任意时间的延迟。然而不幸的是，在调用一个方法之前无法知道其是否阻塞。NIO的channel抽象的一个重要特征就是可以通过配置它的阻塞行为，来实现非阻塞式的信道：

```java
channel.configureBlocking(false)
```

在非阻塞式信道上调用一个方法总是会立即返回。这种调用的返回值指示了所请求的操作完成的程度。例如，在一个非阻塞式ServerSocketChannel上调用accept()方法，如果有连接请求来了，则返回客户端SocketChannel，否则返回null。(这种风格，有点同步非阻塞IO模型)

举一个socket连接的例子，**serve端采用BIO实现，client采用NIO实现**：

### 2.3.1 BIOServer实现

```java
package default_package.NIO练习.SocketChannel练习.server;


import default_package.NIO练习.Common;

import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.SocketAddress;


public class BIOServer {
    public static void main(String[] args) {
        startServer();
    }


    /*
    基于阻塞的socket server端实现
    功能是，打印出client发过来的message


     */
    public static void startServer() {
        ServerSocket serverSocket = null;
        InputStream inputStream = null;


        try {
            serverSocket = new ServerSocket(8080);

            int readLength = 0;
            byte[] receiveBufferBytes = new byte[1024 * 4];

            while (true) {
                //这一步会一直阻塞，直到有了client连接
                Socket client = serverSocket.accept();

                SocketAddress clientAddress = client.getRemoteSocketAddress();
                System.out.println("Handling client at:" + clientAddress);
                inputStream = client.getInputStream();
                while (-1 != (readLength = inputStream.read(receiveBufferBytes))) {
                    System.out.println("received message:" + new String(receiveBufferBytes, 0, readLength));
                }

                //对该client没有多余操作

            }


        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            Common.closeObj(serverSocket);
            Common.closeObj(inputStream);
        }


    }


}

```

### 2.3.2 NIOClient实现

再来看一下Client端实现，注意！！！Client虽然用了NIO的写法，但是并没有使用Selector，本质上还是一种同步阻塞因为，只有finishConnect()为true时，才可以真正开始进行消息发送。

```java
package default_package.NIO练习.SocketChannel练习.client;

import default_package.NIO练习.Common;

import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;
import java.util.concurrent.TimeUnit;

public class NIOClient {
    public static void main(String[] args) {
        startClient();
    }


    /*
    NIO实现的socket client实现

    功能是：
     */

    public static void startClient() {
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024 * 10);
        SocketChannel socketChannel = null;
        try {
            socketChannel = SocketChannel.open();
            socketChannel.configureBlocking(false);

            //如果socketChann是非阻塞模式的，这一步会阻塞，直到连接成功
            socketChannel.connect(new InetSocketAddress(8080));


            /*
             * finishConnect()
             *<p> If this channel is already connected then this method will not block
             * and will immediately return {@code true}.  If this channel is in
             * non-blocking mode then this method will return {@code false} if the
             * connection process is not yet complete.  If this channel is in blocking
             * mode then this method will block until the connection either completes
             * or fails, and will always either return {@code true} or throw a checked
             * exception describing the failure.
             * */
            while (!socketChannel.finishConnect()) {
                System.out.println("NOT connect yet...");
                //就算是 socketChannel.configureBlocking(false);  连接也是需要时间的
                //非阻塞的Channel，调用finishConnect会立即返回状态
                //但是，也只有返回true时，接下来的才可以继继续进行下去
                //这里暂时还未用到selector
            }

            System.out.println("Connect success!");

            int i = 0;
            while (i < 10) {
                TimeUnit.SECONDS.sleep(1);
                String message = "This is NO." + i + " message.";
                byteBuffer.clear();

                //向buffer中写入字节，position会移动
                byteBuffer.put(message.getBytes());

                //调整索引位置，准备让系统读取数据
                byteBuffer.flip();

                while (byteBuffer.hasRemaining()) {
                    System.out.println(byteBuffer);
                    socketChannel.write(byteBuffer);//其实没必要用while，这种写法代表一次性写完

                }

                i++;

            }


        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            Common.closeObj(socketChannel);
        }


    }


}
```
注意！！！！！！！！！！！！！！！

**注意SocketChannel.write()方法的调用是在一个while循环中的。write()方法无法保证能写多少字节到SocketChannel。所以，我们必须！！！！重复调用write()直到Buffer没有要写的字节为止。非阻塞模式下,read()方法在尚未读取到任何数据时可能就返回了。所以需要关注它的int返回值，它会告诉你读取了多少字节。**


### 2.3.4 BIOServer+NIOClient运行效果

分别运行Server和Client主函数。

Server端的打印如下：

```
Handling client at:/192.168.3.6:65217
received message:This is NO.0 message.
received message:This is NO.1 message.
received message:This is NO.2 message.
received message:This is NO.3 message.
received message:This is NO.4 message.
received message:This is NO.5 message.
received message:This is NO.6 message.
received message:This is NO.7 message.
received message:This is NO.8 message.
received message:This is NO.9 message.
```

Client端的打印如下:

```
Connect success!
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
java.nio.HeapByteBuffer[pos=0 lim=21 cap=10240]
```

Client一上来就直接Connect成功，是因为，本机直连，十分迅速，0延迟。非本机直连的情况下，必定会打印"NOT connect yet..."。

### 2.3.5 NIOServer
该节使用NIO实现SocketServer。

到目前为止，所举的案例中都没有涉及Selector。Selector类可以用于避免使用阻塞式客户端中很浪费资源的“忙等”方法。例如，考虑一个IM服务器。像QQ或者旺旺这样的，可能有几万甚至几千万个客户端同时连接到了服务器，但在任何时刻都只是非常少量的消息（非资源密集型的长连接）。需要读取和分发。这就需要一种方法阻塞等待，直到至少有一个信道可以进行I/O操作，并指出是哪个信道。如果使用同步阻塞IO模型，千万的连接可能会导致千万的个Server端的线程等待 或者 线程的新建、销毁，为了避免这种线程资源消耗，**考虑使用NIO的多路复用模型，减小多线程同时阻塞的压力。**

NIO的选择器就实现了这样的功能。一个Selector实例可以同时检查一组/多个信道的I/O状态。用专业术语来说，选择器就是一个多路开关选择器，因为一个选择器能够管理多个信道上的I/O操作。然而如果用传统的方式来处理这么多客户端，编程人员使用的方法是循环地一个一个地去检查所有的客户端是否有I/O操作，如果当前客户端有I/O操作，则可能把当前客户端扔给一个线程池去处理，如果没有I/O操作则进行下一个轮询，当所有的客户端都轮询过了又接着从头开始轮询；这种方法是非常笨而且也非常浪费资源，因为大部分客户端是没有I/O操作，编程人员也需要使用循环去检查。

而Selector就不一样了，它在内部可以同时管理多个I/O，当一个信道有I/O操作的时候，他会通知Selector，Selector就是记住这个信道有I/O操作，并且知道是何种I/O操作，是读呢？是写呢？还是接受新的连接；而编程人员只需要直接调用Selector的方法，就能直接获取已经就绪的信道，相当于轮询操作已在Selector内部实现了。所以如果使用Selector，它返回的结果只有两种结果，一种是0，即在你调用的时刻没有任何客户端需要I/O操作，另一种结果是一些需要I/O操作的客户端。对于编程人员来说，这样一种通知的方式比那种主动轮询的方式要高效得多！轮询的实现已在底层已经实现。

要使用选择器（Selector），需要创建一个Selector实例（使用静态工厂方法open()得到Selector实例），并将Channel注册（register）到Selector实例中。最后，调用选择器的select()方法获取就绪的信道，select()方法导致阻塞，直到有一个或更多的信道准备好了I/O操作或等待超时。select()方法将返回可进行I/O操作的信道数量。现在，在一个单独的线程中，通过调用select()方法就能检查多个信道是否准备好进行I/O操作。如果经过一段时间后仍然没有信道准备好，select()方法就会返回0，并允许程序继续执行其他任务。

```java
package default_package.NIO练习.SocketChannel练习.server;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.ServerSocketChannel;
import java.nio.channels.SocketChannel;
import java.util.Iterator;


/*
要点：
1、readLength = socketChannel.read(byteBuffer)，-1表示channel已关闭，>0表示通道没关且读到了字节；
通道已经关闭时，务必cancel当前selectionKey，否则会死循环，因为selectionKey.isReadable()为true并不代表通道没有关闭！！

2、selectionKey被cancel后，还去调用isAcceptable之类的操作，会抛出已经取消的异常；因此，必须使用if  else 来保证情况之间是互斥的，
从而避免，前面的操作已经取消了，下面仍然去判断从而抛出异常

 */

public class NIOServer {
    public static void main(String[] args) {
        startServer();
    }


    private static final int BUF_SIZE = 1024 * 4;
    private static final int PORT = 8080;
    private static final int Selector_TIMEOUT = 3000;

    private static void startServer() {


        Selector selector = null;
        ServerSocketChannel serverSocketChannel = null;


        try {

            selector = Selector.open();
            serverSocketChannel = ServerSocketChannel.open();
            serverSocketChannel.bind(new InetSocketAddress(8080));//绑定/启动的socket服务端口8080
            serverSocketChannel.configureBlocking(false);//配置了非阻塞才能使用Selector


            //重点来了！！！！！！！！！！！！！！！！！
            //在多路监控中注册当前的信道，就绪的时间类型是 SelectionKey.OP_ACCEPT
            //即，这个信道上发生 SelectionKey.OP_ACCEPT  时，则认为其信道已经就绪
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

            while (true) {

                //This method performs a blocking
                if (0 == selector.select(Selector_TIMEOUT)) {
                    System.out.println(Selector_TIMEOUT + " no channel ready");
                    continue;
                }

                System.out.println("就绪个数：" + selector.selectedKeys().size());

                //获取就绪的信道集合
                Iterator<SelectionKey> selectionKeyIterator = selector.selectedKeys().iterator();


                while (selectionKeyIterator.hasNext()) {
                    SelectionKey selectionKey = selectionKeyIterator.next();

                    if (selectionKey.isAcceptable()) {
                        handleAccept(selectionKey);
                    } else if (selectionKey.isConnectable()) {
                        handleConnect(selectionKey);
                    } else if (selectionKey.isReadable()) {
                        handleRead(selectionKey);
                    } else if (selectionKey.isWritable()) {
                        handleWrite(selectionKey);
                    }

                    selectionKeyIterator.remove();
                    // 当前的获取accept和read的channel已经remove过了
                    // 但是！！！！channel的read状态确实一直有的，handleRead(selectionKey);  这一句会一直执行，造成死循环

                }


                System.out.println("就绪个数：" + selector.selectedKeys().size());


            }


        } catch (IOException e) {
            e.printStackTrace();
        }


    }

    private static void handleWrite(SelectionKey selectionKey) {
        System.out.println("Start to handle selectionKey.isWritable()");

        try {
            SocketChannel sc = (SocketChannel) selectionKey.channel();

            ByteBuffer buf = (ByteBuffer) selectionKey.attachment();
            buf.flip();
            while (buf.hasRemaining()) {
                sc.write(buf);
            }

            //虽然前面已经使用了while循环进行读取，到这里的还是要压缩一下
            buf.compact();

        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    private static void handleRead(SelectionKey selectionKey) {
        System.out.println("Start to handle selectionKey.isReadable()");

        //取出channel
        SocketChannel socketChannel = (SocketChannel) selectionKey.channel();
        //取出附加的对象
        ByteBuffer byteBuffer = (ByteBuffer) selectionKey.attachment();

        try {

            byteBuffer.clear();
            int readLength = 0;

            //socketChannel.read(byteBuffer)  会向Buffer中写入数据
            while ((readLength = socketChannel.read(byteBuffer)) > 0) {
                System.out.println("readLength:" + readLength);

                byteBuffer.flip();//更新position位置，为读取做好准备

                while (byteBuffer.hasRemaining()) {
                    System.out.print((char) byteBuffer.get());
                }

                byteBuffer.clear();


            }

            if (-1 == (readLength = socketChannel.read(byteBuffer))) {
                System.out.println("收到Client的Close()指令！取消可读监听！");

                selectionKey.cancel();
            }


        } catch (IOException e) {
            e.printStackTrace();
        } finally {
//            Common.closeObj(socketChannel);
        }


    }

    private static void handleConnect(SelectionKey selectionKey) {
        System.out.println("Start to handle selectionKey.isConnectable()");
    }

    private static void handleAccept(SelectionKey selectionKey) {
        System.out.println("Start to handle selectionKey.isAcceptable()");

        ServerSocketChannel serverSocketChannel = (ServerSocketChannel) selectionKey.channel();
        try {
            SocketChannel socketChannel = serverSocketChannel.accept();
            System.out.println("Start to handle client:" + socketChannel.getRemoteAddress());

            socketChannel.configureBlocking(false);

            //将Client通道的可读状态进行注册，并且附加一个缓存对象，用于处理
            //因此，还要额外实现，对可读 SelectionKey  的处理
            socketChannel.register(selectionKey.selector(), SelectionKey.OP_READ, ByteBuffer.allocate(BUF_SIZE));

        } catch (IOException e) {
            e.printStackTrace();
        }


    }

}

```
**要点：**

1、readLength = socketChannel.read(byteBuffer)，-1表示channel已关闭，>0表示通道没关且读到了字节；
通道已经关闭时，务必cancel当前selectionKey，否则会死循环，因为selectionKey.isReadable()为true并不代表通道没有关闭！！

2、selectionKey被cancel后，还去调用isAcceptable之类的操作，会抛出已经取消的异常；因此，必须使用if  else 来保证情况之间是互斥的，
从而避免，前面的操作已经取消了，下面仍然去判断从而抛出异常

3、注意SocketChannel的关闭时机，后续还有读写操作就不能提前关闭！！！客户端主动关闭时，可作为server关闭的标志！
```java
int num = channel.read(bytebuffer);
if(num == -1){
    selectionKey.cancel();
    channel.close();
}
```



#### 2.3.5.1 讲解

以上实现其实完全使用了NIO的特性，由BIO向前跨了两步：

1、由   面向流  变为  面向缓存

2、多线程的同步阻塞  变为   多路复用的“非阻塞”

先讲如果只进行面两流到面向缓存的转换，ServerSocket应该如下实现：

```java
//打开一个SocketServerChanne
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
//设置监听的端口
serverSocketChannel.socket().bind(new InetSocketAddress(9999));

//======================================
//如果设置成阻塞模式！！！！！监听进来的连接
while(true){
    //该accept会阻塞住
    SocketChannel socketChannel = serverSocketChannel.accept();
    // do something with socketChannel...
}
//======================================
//如果设置成  非  阻塞模式！！！！！
serverSocketChannel.configureBlocking(false); 
//非阻塞模式，其实还是要等待连接可用 或者  accept到一个client
while (true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    if (socketChannel != null)
    {
        // do something with socketChannel...
    }
}

//关闭连接
```

第二大步的Selector的使用和介绍放到下一节。

## 2.4 Selector

### 2.4.1 创建选择器

Selector的创建：Selector selector = Selector.open();

### 2.4.2 选择器中注册感兴趣事件

为了将Channel和Selector配合使用，必须将Channel注册到Selector上，通过SelectableChannel.register()方法来实现，沿用案例5中的部分代码：

```
ServerSocketChannel ssc= ServerSocketChannel.open();

ssc.socket().bind(new InetSocketAddress(PORT));

ssc.configureBlocking(false);

//注册该通道感兴趣的事件
ssc.register(selector, SelectionKey.OP_ACCEPT);
```

与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。FileChannel不能注册到选择器中是因为，register()是SelectableChannel的方法，而FileChannel没有继承/是实现SelectableChannel。

SelectableChannel.register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：

1、连接就绪事件，用int 常量 SelectionKey.OP_CONNECT 表示

2、接收就绪事件，用int 常量 SelectionKey.OP_ACCEPT 表示

3、读就绪事件，用int 常量 SelectionKey.OP_READ 表示

4、写就绪事件，用int 常量 SelectionKey.OP_WRITE 表示

### 2.4.3 获取就绪的通道

```java
//This method performs a blocking
if (0 == selector.select(Selector_TIMEOUT)) {
    System.out.println("within "Selector_TIMEOUT + ", no channel ready");
    continue;
}

System.out.println("就绪个数：" + selector.selectedKeys().size());

//获取就绪的信道集合
Iterator<SelectionKey> selectionKeyIterator = selector.selectedKeys().iterator();
```

一旦向Selector注册了一或多个通道，就可以调用几个重载的select()方法。这些方法返回你所感兴趣的事件（如连接、接受、读或写）已经准备就绪的那些通道。可以为四种就绪事件分别新建4个Selector，对应的事件分别注册到其中。

下面是select()及其几种重载方法：

- int select()，**阻塞到至少有一个通道在你注册**的事件上就绪了。
- int select(long timeout)，select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)。
- int selectNow()，selectNow()不会阻塞，不管什么通道就绪都立刻返回。此方法执行非阻塞的选择操作。如果自从前一次选择操作后，没有通道变成可选择的，则此方法直接返回零。



select()方法返回的int值表示有多少通道已经就绪。亦即，自**上次调用select()方法后有多少通道变成就绪状态**。如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。**每次select()调用只返回状态变为就绪的通道数。**

然后可以通过调用selector的selectedKeys()方法，访问“已选择键集（selected key set）”中的就绪通道。如下所示：

```
Set<SelectionKey> selectedKeys = selector.selectedKeys();
```

当向Selector注册Channel时，Channel.register()方法会返回一个SelectionKey 对象。这个对象代表了注册到该Selector的通道。

**注意每次迭代末尾的keyIterator.remove()调用。Selector不会自己从已选择键集中移除SelectionKey实例。必须在处理完通道时自己移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中。**

SelectionKey.channel()方法返回的通道需要转型成你要处理的类型，如ServerSocketChannel或SocketChannel等。

# 3 NIO 内存映射文件





# 4 其余功能介绍