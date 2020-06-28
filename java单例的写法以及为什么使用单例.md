@[TOC]
# 1 为什么要用单例模式
单例模式是一种对象创建模式，用于生产一个对象的实例，它可以确保系统中一个类只产生一个实例，这样做有两个好处:
1. 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销。
2. 由于new操作的次数减少，所以系统内存的使用评率也会降低，这将减少GC压力，缩短GC停顿时间。
由于以上两点可知单例模式的使用对于系统的关键组件和频繁使用的对象来说是可以有效的改善系统的性能的。
单例的核心是通过一个方法返回唯一的一个对象实例，首先单例类必须有一个private访问级别的构造函数，因为，只有这样，才能保证单例不会在系统中的其他代码内被实例化，其次，instance成员变量和getInstance方法必须是static的。

## 1.1 单例vs静态类vs@service
参考<https://blog.csdn.net/jeason_chan_zju/article/details/99119309>
# 2 单例的几种写法
## 2.1 饿汉法
```java
public class Hungry_man_pattern {
    private Hungry_man_pattern(){
        System.out.println("调用"+this.getClass()+"饿汉模式构造函数!");
    }

    private static Hungry_man_pattern singleton=new Hungry_man_pattern();

    public static Hungry_man_pattern getSingleton(){
        return singleton;
    }
}
```
缺点很明显，即使没有任何饿汉模式调用，也会在程序一开始就会完成类中静态对象的初始化，因此，一开始就会打印"调用饿汉模式构造函数!"，如果这个类很庞大，那么则会非常占用内存，为了完成完善这个情况，得到懒汉模式，只在第一次使用单例的时候进行初始化。
## 2.2 懒汉法
```java
public class Lazy_man_pattern {
    /*
    饿汉模式,类还没有使用前就创建进内存
     */
    private Lazy_man_pattern() {
        System.out.println("调用" + this.getClass() + "的构造函数!");
    }

    private static Lazy_man_pattern singleton = null;

    public static Lazy_man_pattern GetSingleton() {
        if (singleton == null) {
            singleton = new Lazy_man_pattern();
            System.out.println("完成懒汉模式单例的初次初始化!");
        }
        return singleton;
    }

}
```
懒汉模式，一定程度上避免了程序启动时大量内存消耗大的问题，一定程度上缓解了程序启动慢的问题，但是如果有多个线程同时第一次调用GetSingleton（）方法，因为不是判断静态单例实例是否为空不是原子操作，可能有多个线程同时对静态单例进行赋值操作，极有可能带来线程安全问题，因此，要解决懒汉模式线程安全问题。
## 2.3 简单线程安全懒汉模式
线程不安全的点在于，判断是都为空，然后进行相关操作，因此，将GetSingleton（）变为同步方法即可解决线线程安全问题。
```java
public class simple_thread_safe_lazy_man_pattern {
    private static simple_thread_safe_lazy_man_pattern singleton = null;

    private simple_thread_safe_lazy_man_pattern() {
        System.out.println("调用了" + this.getClass() + "简单线程懒汉模式构造函数！");
    }

    public static synchronized simple_thread_safe_lazy_man_pattern getSingleton() {
        //由于同步方法的资源比同步快的资源消耗大，因此，可以考虑进一步缩小同步块范围
        if (singleton == null) {
            singleton = new simple_thread_safe_lazy_man_pattern();
        }
        return singleton;
    }
}
```
当第一次使用获得静态实例时，要对静态实例进行初始化，这时候，只需要有里面一层带同步的判断即可；如果，不是第一次想获得静态单例，则不带同步的if发挥作用，不走同步块，能实现很好的并发度。因此，此种带双重检查锁的方法，实现了初始化时的synchronized同步，和非初始化时的完全并发，看上去十分完美！
但是，其实还是存在线程安全问题！！！！假设线程A执行到了第一个“iif (singleton == null) ”，它判断对象为空，于是线程A执行到“  singleton = new simple_thread_safe_lazy_man_pattern();”去初始化这个对象，首先赋予这个引用对象地址，然后再在内存中获取内存块初始化对象，获取内存再写入内存也需要时间，如果此时（singleton已经不是null并且指向一个不完整的对象）线程B也执行到了第一个“if (null == singObj )”，它判断不为空，于是直接跳到“return singObj”得到了这个对象。但是，这个对象还没有被完整的初始化！得到一个没有初始化完全的对象并没有用！
## 2.4 静态内部类法（initialization on Demand Holder）
```java
public class inner_static_class_pattern {
    private static class SingetonHoler{
        public final static inner_static_class_pattern singleton=new inner_static_class_pattern();
    }

    private inner_static_class_pattern(){
        System.out.println("调用"+this.getClass()+"构造函数！");
    }

    public static inner_static_class_pattern getSingleton(){
        return SingetonHoler.singleton;
    }
}
```
这种方法使用内部类来做到延迟加载对象，因为java机制规定，内部类SingletonHolder只有在getSingleton()方法第一次调用的时候才会被加载（实现了lazy，而懒汉模式的单例实例是个实例对象，一load就直接初始化了），而且其加载过程是线程安全的（java自己的实现是线程安全的）。内部类加载的时候实例化一次instance。这种写法最大的美在于，完全使用了Java虚拟机的机制进行同步保证，没有一个同步的关键字。

