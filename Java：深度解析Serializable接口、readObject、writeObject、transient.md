# 1 前言
参考文章：

JAVA对象流序列化时的readObject，writeObject，readResolve是怎么被调用的   https://blog.csdn.net/u014653197/article/details/78114041

Java Serializable：明明就一个空的接口嘛   https://blog.csdn.net/qing_gee/article/details/93176705

常见的代码静态扫描工具，对于实现了Serializable接口，但没有定义readObject和writeObject方法的类，经常报告警。深度解析一下的为什么定义这个private的方法。

# 2 实践与源码解析

## 2.1 正常的(反)序列化步骤
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

## 2.2 writeObject和readObject
当一个类实现了Serializable接口时，静态代码扫描工具认为我们应该为这个类定义writeObject和readObject方法，一般都是无脑加下面的代码：

```java
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();
    }
```

对于普通的类，一般没有问题。因此，要来了解一下序列化和反序列化的具体流程，由于两个过程差不多，以HashSet及其readObejct过程进行分析。

## 2.3 HashSet源码
先来看一下HashSet中和序列化相关的源码：

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
{
    static final long serialVersionUID = -5024744406713321676L;

    private transient HashMap<E,Object> map;

    //================定义并实现了writeObject，用的不是defaultWriteObject=============

    /**
     * Save the state of this {@code HashSet} instance to a stream (that is,
     * serialize it).
     *
     * @serialData The capacity of the backing {@code HashMap} instance
     *             (int), and its load factor (float) are emitted, followed by
     *             the size of the set (the number of elements it contains)
     *             (int), followed by all of its elements (each an Object) in
     *             no particular order.
     */
    private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // Write out any hidden serialization magic
        s.defaultWriteObject();

        // Write out HashMap capacity and load factor
        s.writeInt(map.capacity());
        s.writeFloat(map.loadFactor());

        // Write out size
        s.writeInt(map.size());

        // Write out all elements in the proper order.
        for (E e : map.keySet())
            s.writeObject(e);
    }

    //================定义并实现了readObject，用的不是defaultReadObject=============
    /**
     * Reconstitute the {@code HashSet} instance from a stream (that is,
     * deserialize it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        // Read in any hidden serialization magic
        s.defaultReadObject();

        // Read capacity and verify non-negative.
        int capacity = s.readInt();
        if (capacity < 0) {
            throw new InvalidObjectException("Illegal capacity: " +
                                             capacity);
        }

        // Read load factor and verify positive and non NaN.
        float loadFactor = s.readFloat();
        if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
            throw new InvalidObjectException("Illegal load factor: " +
                                             loadFactor);
        }

        // Read size and verify non-negative.
        int size = s.readInt();
        if (size < 0) {
            throw new InvalidObjectException("Illegal size: " +
                                             size);
        }

        // Set the capacity according to the size and load factor ensuring that
        // the HashMap is at least 25% full but clamping to maximum capacity.
        capacity = (int) Math.min(size * Math.min(1 / loadFactor, 4.0f),
                HashMap.MAXIMUM_CAPACITY);

        // Constructing the backing map will lazily create an array when the first element is
        // added, so check it before construction. Call HashMap.tableSizeFor to compute the
        // actual allocation size. Check Map.Entry[].class since it's the nearest public type to
        // what is actually created.
        SharedSecrets.getJavaObjectInputStreamAccess()
                     .checkArray(s, Map.Entry[].class, HashMap.tableSizeFor(capacity));

        // Create backing HashMap
        map = (((HashSet<?>)this) instanceof LinkedHashSet ?
               new LinkedHashMap<>(capacity, loadFactor) :
               new HashMap<>(capacity, loadFactor));

        // Read in all elements in the proper order.
        for (int i=0; i<size; i++) {
            @SuppressWarnings("unchecked")
                E e = (E) s.readObject();
            map.put(e, PRESENT);
        }
    }

}
```

"private transient HashMap<E,Object> map;"这个私有变量就是用来保存节点信息的，HashSet和LinkedHashSet底层就是用HashMap实现的，CopyOnWriteArraySet底层是用CopyOnWriteArrayList实现的，私有的map**被用transient关键字修饰了，来表明序列化的时候跳过该变量**。

存储的信息是保存在私有变量map里面的，但是，这个map又是在序列化时忽略的，那为什么HashSet还能在序列化和反序列化后还原里面的信息呢？因为HashSet定义并实现了readObject方法与writeObject方法。

### 2.3.1 HashSet的writeObject方法
采用串行的方式，按顺序做了以下几个的几件事：

1. s.defaultReadObject();直接使用默认的反序列化，这个时候s中并没有记录私有变量map的任何相关信息，因为是transient的
2. 向s中补充写入，map的的容量
3. 向s中补充写入，map的负载因子
4. 向s中补充写入，map的实际数量
5. 以循环的方式，向map中逐个添加Set中的元素

可见，通过实现writeObject方法，做了额外信息的保存工作

### 2.3.2 HashSet的readObject方法

由于，write的时候以什么顺序write的，read就必须以什么顺序读，因此，99.99999%的可能肯定和上面的顺序相同，看源码分析发现，write的步骤是：

1. s.defaultWriteObject();和前面的对应，先使用默认的序列化
2. 读取的map的容量，做一下校验
3. 读取map的负载因子，做一下校验
4. 读取的map的实际数量，做一下校验

5. 根据上面读出来的值，重新分配一下容量，以保证适当的负载率；
6. 做一些什么检查……构造一个map

7. 根据size大小，将Set存放的元素一个一个读出来，放进map

可见，通过实现readObject方法，对原先的对象做了复原，但是！！！容量有可能已经变化了，因为第5步做了重新计算。


### 2.3.3 小结
可见，有些被transient修饰的变量不想被序列化，但是，里面又包含一些有用的信息时，可自行定制readObject和writeObject方法，来向对象字节流中补充写入/读取 一些信息。

一般情况下，可以直接使用default的即可。

# 2.4 readObject和writeObject如何被调用
