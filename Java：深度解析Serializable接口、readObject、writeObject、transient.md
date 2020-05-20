# 1 前言
参考文章：

JAVA对象流序列化时的readObject，writeObject，readResolve是怎么被调用的   https://blog.csdn.net/u014653197/article/details/78114041

Java Serializable：明明就一个空的接口嘛   https://blog.csdn.net/qing_gee/article/details/93176705

常见的代码静态扫描工具，对于实现了Serializable接口，但没有定义readObject和writeObject方法的类，经常报告警。深度解析一下的为什么定义这个private的方法。

# 2 实践与源码解析

## 2.1 正常的序列化和反序列化
以序列化和反序列化 String 类为例：

```java
package default_package.序列化接口练习;

import java.io.*;

public class Main {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        //以覆盖写入的的方式发开一个本地文件
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(new FileOutputStream("test0520.txt"));

        //连续写入两个对象
        objectOutputStream.writeObject("字符串1");
        objectOutputStream.writeObject("字符串2");

        //流的关闭要规范，尽量在finally中进行关闭，此处不是代码的重点
        objectOutputStream.close();

        //==============================================


        ObjectInputStream objectInputStream = new ObjectInputStream(new FileInputStream("test0520.txt"));
        //连续读出两个对象
        System.out.println(objectInputStream.readObject());
        System.out.println(objectInputStream.readObject());

        //再次尝试读，会报错
        System.out.println(objectInputStream.readObject());
        /*
        报错信息如下：
        Exception in thread "main" java.io.EOFException
        at java.io.ObjectInputStream$BlockDataInputStream.peekByte(ObjectInputStream.java:2950)
        at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1534)
        at java.io.ObjectInputStream.readObject(ObjectInputStream.java:427)
        at default_package.序列化接口练习.Main.main(Main.java:17)

        因为检测读到了文件、流的末尾，上报了EOF错误。多个类实例之前的分割符可能也是通过类似的读特定字节进行区分实现的，
        详见JDK源码：
        D:/Program Files (x86)/Java1.8/src.zip!/java/io/ObjectInputStream.java:1533   byte tc; 这个变量

        这种挨个读字节流的处理方式，可以借鉴使用到 多路复用中的 ByteBuffer的字节读取

        */

    }

}
```

## 2.2 
