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

举一个socket连接的例子，serve端采用BIO实现，client采用NIO实现：

先看server实现：

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



## 2.4 Selector