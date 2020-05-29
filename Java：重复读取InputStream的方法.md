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

在InputStream读取的时候，会有一个pos指针（position指针），他指示每次读取之后下一次要读取的起始位置，当读到最后一个字符的时候，pos指针不会重置。

看一下
