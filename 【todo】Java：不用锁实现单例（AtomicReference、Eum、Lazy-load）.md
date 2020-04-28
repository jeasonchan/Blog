# 1 前言
阿里面试官：如何实现一个线程安全的单例，前提是不能加锁   https://www.jianshu.com/p/f3fae8658f13


文章要点：
1、几种过时的单例写法
2、利用lazy-loard的机制，使用内部静态类保证只初始化一次
3、使用Eum实现单例
4、利用CAS保证对象只被初始化一次

# 2 几种写法及分析
# 2.1 内部静态类
```java
public class Singleton {

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();

    }

    private Singleton (){}

    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;

    }

}
```
使用了lazy-loading,Singleton类被装载了，但是，内部静态类的成员变量instance并没有立即初始化。因为SingletonHolder类没有被主动使用，只有第一次调用时，内部静态类才会进行初始化，即显式调用getInstance方法时，才会显式装载SingletonHolder类，从而实例化instance。


# 2.2 枚举写法
```java
public enum Singleton {
    INSTANCE;

    private Singleton(){
        //枚举类的默认无参构造方法
    }
}
```

这种方式是Effective Java作者Josh Bloch 提倡的方式，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象，可谓是很坚强的壁垒。

# 2.3 自旋锁（CAS）
以上几种答案，其实现原理都是利用借助了类加载的时候初始化单例。即借助了ClassLoader的线程安全机制。

所谓ClassLoader的线程安全机制，就是ClassLoader的loadClass方法在加载类的时候使用了synchronized关键字。也正是因为这样， 除非被重写，这个方法默认在整个装载过程中都是同步的，也就是保证了线程安全。

所以，以上各种方法，虽然并没有显示的使用synchronized，但是还是其底层实现原理还是用到了synchronized。

CAS是乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。实现单例的方式如下：

```java
public class Singleton {

    //使用原子引用容器封装待初始化的单例
    private static final AtomicReference<Singleton> INSTANCE = new AtomicReference<>();

    private Singleton() {}//无参构造方法

    public static Singleton getInstance() {
        while(true) {
            Singleton singleton = INSTANCE.get();

            if (null != singleton) {
                return singleton;
            }

            //建议放到while循环外，只创建一次，while内只进行CAS
            singleton = new Singleton();

            if (INSTANCE.compareAndSet(null, singleton)) {
                return singleton;

            }

        }

    }

}

```
用CAS的好处在于不需要使用传统的锁机制来保证线程安全,CAS是一种基于忙等待的算法,依赖底层硬件的实现,相对于锁它没有线程切换和阻塞的额外消耗,可以支持较大的并行度。

CAS的一个重要缺点在于如果忙等待一直执行不成功(一直在死循环中,所有的线程都无法成功完成CAS种的Set操作),会对CPU造成较大的执行开销。

另外，如果大量线程同时执行到singleton = new Singleton();的时候，会有大量对象同时创建，很可能导致内存溢出。同时，由于用于赋值的singleton = new Singleton();对象，写在了while循环中，就算是同一个线程也可能反复new对象，因此，可以将singleton = new Singleton();放到while外面，只创建一次，while内只进行CAS操作。
