# 1 前言

参考文章：

重复读取InputStream的方法    https://blog.csdn.net/u014656173/article/details/77280705


进行MyBatis学习时，SqlSessionFactoryBuilder需要使用转换为流的配置文件。

但是，我将配置文件  getResourceAsStream  后，先readAllBytes，将配置文件打印了出来，然后再给SqlSessionFactoryBuilder构造对象。但是，报了EOF的错误，应该不是文件末尾的地方，却读到Byte标志位——文件末尾，报了该异常。

这个异常让我想起了，writeObject和readObject方法，向输出流中连续写入两个对象，但是读的时候尝试读三个对象，在第三次调用readObject时就会报错，同样的EOF异常。

再结合，NIO中的ByteBuffer中的书签、pos、limit的机制，推测InputStream也有类似的pos变量，用来标记读到到哪个字节了。

下面解析一下的为什么InputStream不能重复读取，和如何实现对InputStream的重复读取。

# 2 为什么设计为不能重复读取

“InputStream就类比成一个杯子，杯子里的水就像InputStream里的数据，你把杯子里的水拿出来了，杯子的水就没有了，InputStream也是同样的道理。”
比喻的非常好，让我们从直观上认识了InputStream为什么不能重复被读。

从更深的代码角度去分析：

在InputStream读取的时候，会有一个pos指针（position指针），他指示每次读取之后下一次要读取的起始位置，当读到最后一个字符的时候，pos指针不会重置,还是指向一个不存在的位置。

看一下几种InputStream的read方法里的关于pos的代码片段：

```java
//BufferedInputStream代码片段：  
 public synchronized int read() throws IOException {  
        if (pos >= count) {  
            fill();  
            if (pos >= count)  
                return -1;  
        }  
        return getBufIfOpen()[pos++] & 0xff;  
    }  

//FileInputStream代码片段：  
public native int read() throws IOException;
```

Java 的List内部是使用数组实现的，遍历的时候也有一个pos指针。但是没有说List遍历一个第二次遍历就没有了，List是可以重复读取的，因为，第二次遍历是创建新的Iterator，所以pos也回到了数组起始位置。对于某些InputStream当然可以也这么做。例如：ByteArrayInputStream ，ByteArrayInputStream就是将一个Java的byte数组保存到对象里，然后读取的时候遍历该byte数组。

ByteArrayInputStream的构造函数和字节读取源码：

```java
public ByteArrayInputStream(byte buf[]) {  
        this.buf = buf;  
        this.pos = 0;  
        //假设长度是10，能取的索引是[0,9]
        this.count = buf.length;  
}  

//读取下一个的字节
public synchronized int read() {  
    //pos++，先使用再自增
    return (pos < count) ? (buf[pos++] & 0xff) : -1;  
}  
```

从ByteArrayInputStream源码来看，要实现重复读取是很简单的，用一个方法重置reset即可，但是为什么没有。我想是为了遵循InputStream的统一标准，也就是达到末尾后，必须是-1，也就是pos超出索引最大值后返回-1。

```java
/** 
     * Reads the next byte of data from the input stream. The value byte is 
     * returned as an <code>int</code> in the range <code>0</code> to 
     * <code>255</code>. If no byte is available because the end of the stream 
     * has been reached, the value <code>-1</code> is returned. This method 
     * blocks until input data is available, the end of the stream is detected, 
     * or an exception is thrown. 
     * 
     * <p> A subclass must provide an implementation of this method. 
     * 
     * @return     the next byte of data, or <code>-1</code> if the end of the 
     *             stream is reached. 
     * @exception  IOException  if an I/O error occurs. 
     */  
    public abstract int read() throws IOException;  
```

InputStream顾名思义就是一个单向的字节流，跟水流一样，要想再次使用就自己再去源头取一下。

InputStream其实不像杯子，更像是一根水管，要想喝水了，就在把水管架在水源与杯子之间，让水流到杯子里（注意：这个动作完成了之后水管里面就没有水了）。

这样看来，**InputStream其实是一个数据通道，只负责数据的流通，并不负责数据的处理和存储等其他工作范畴。**
但是，其实有的InputStream实现类是可以实现数据的处理工作的。但是没有这么做，这就是规范和标准的重要性。

#3 重复读取InputStream的方法