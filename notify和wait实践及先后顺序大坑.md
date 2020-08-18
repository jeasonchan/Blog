@[TOC]
# 1 概述
想练习一下notify和wait，没想到踩了坑。假设monitor对象是object这个实例，先object.wait()，后object.notifyAll()，由于wait()是阻塞方法，根本执行不到object.notifyAll()。因此，必须，先object.notifyAll()，后object.wait()。
# 2 notify和wait介绍
object.wait() —— 暂停一个线程，同时，让出object的控制权，
object.notify() —— 唤醒随机的/或者优先级更高的一个，以object为锁对象的线程，被唤醒的线程有机会去竞争object的控制权，
object.notifyAll() —— 唤醒**所有的**，以object为锁对象的线程，让他们去竞争object的控制权。
想要使用这两个方法，我们需要先有一个对象 Object吗，可以是任何类实例甚至一整个类。在多个线程之间，我们可以通过调用同一个对象的wait()和notify()来实现不同的线程间的可见，即对这个对象的控制权。**只有拥有这个对象控制权（monitor）的线程，才用使用wait和notify方法！**
注意，sleep并不会释放对象控制权，只有wait才会。notify只是通知已经处于wait状态的线程，可以竞争控制权了，是否竞争到控制权，看资源调度和程序员的代码安排。
## 2.1 对象控制权（monitor）
参考了<https://segmentfault.com/a/1190000018096174>。
线程安全中，考虑的就是对象的控制权问题，即增、删、改同一时间最好只能有一个能够执行，对象的控制权（monitor）就是通过锁来控制的，只有一个主线程的程序中，任何对象的控制权必定都是主线程的！如果有以下代码：
```java
public class ThreadTest {
    public static void main(String[] args) {
        Object object = new Object();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    object.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```
运行时必然报错，java.lang.IllegalMonitorStateException，因为object的控制权默认在主线程手中，new出来的线程没有无法通过object进行wait，因此，要获得对象控制权，synchronized(object)是一种方法，因此，做出以下修改：
```java
public class ThreadTest {
    public static void main(String[] args) {
        Object object = new Object();
        new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (object){ // 修改处
                    try {
                        object.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();
    }
}
```
这样就能保证new出来的线程能够从主线程中拿到控制权，该线程wait后，obejct对象又对主线程可见了，主线程可以对其进行一些操作了。
# 3 先wait再notifyAll
直接看源码：
```java
package com.xxx.exercise_notify_wait;

import java.util.Queue;
import java.util.concurrent.LinkedBlockingQueue;

public class Main {
    public static void main(String[] args) {
        Main main = new Main();
        new Thread(main::producer, "producer thread").start();
        new Thread(main::consumer, "consumer thread").start();
        System.out.println("主线程执行到最后一行！");
    }

    //----------------------------------------
    private static final Queue<String> stringList = new LinkedBlockingQueue<>();

    /*
     以stringList为monitor，一次往其中添加一条信息，直到加到五条，然后将所锁对象让给消费者
     */
    public void producer() {
        while (true) {
            System.out.println("生产者 进入 while 循环第一行");
            synchronized (stringList) {

                while (stringList.size() < 5) {
                    String temp = System.currentTimeMillis() + Util.getFiveRandomChar();
                    System.out.println(Thread.currentThread().getName() + " : " + "producer will add: " + temp);
                    stringList.add(temp);

                }

                try {
                    stringList.wait();//加满了，让出monitor，自身处于blocking状态
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                stringList.notifyAll();//通知其他以stringList为锁的同步块，可以竞争锁资源了
                System.out.println("生产者 同步块 最后一行");
            }

            System.out.println("生产者 while循环 最后一行");
        }
    }

    /*
    消费者，取得锁之后就去消费，打印相应的值并且将值删除
     */
    public void consumer() {
        while (true) {
            System.out.println("消费者 进入 while 循环第一行");
            synchronized (stringList) {

                while (!stringList.isEmpty()) {
                    System.out.println(Thread.currentThread().getName() + " : " + "consumer will remove: " + stringList.peek());
                    stringList.remove();
                }

                try {
                    stringList.wait();//全部消费完毕，让monitor资源
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                stringList.notifyAll();//通知其他的线程可以竞争monitor资源了

                System.out.println("消费者 同步块 最后一行");
            }

            System.out.println("生产者 while循环 最后一行");
        }
    }

    /*
    wait和notify必须在同步块中，并且，必须通过被monitor的对象进行调用！
     */
}
```
执行结果：
```shell
主线程执行到最后一行！
生产者 进入 while 循环第一行
producer thread : producer will add: 1564536449796iruwk
producer thread : producer will add: 1564536449811mrdsm
producer thread : producer will add: 1564536449813fqzrq
producer thread : producer will add: 1564536449813zysif
producer thread : producer will add: 1564536449813tlvlf
消费者 进入 while 循环第一行
consumer thread : consumer will remove: 1564536449796iruwk
consumer thread : consumer will remove: 1564536449811mrdsm
consumer thread : consumer will remove: 1564536449813fqzrq
consumer thread : consumer will remove: 1564536449813zysif
consumer thread : consumer will remove: 1564536449813tlvlf
#一直阻塞住了，和预计的生产者继续加字符串结果不一样……
```
经过调试，发现，生产者和消费者线程全都处于wait状态，被阻塞了，经过分析，原来是**wait()是阻塞方法，根本执行不到object.notifyAll()**，被自己蠢哭了……
# 4 先notifyAll在wait
```java
package com.xxx.exercise_notify_wait;

import java.util.Queue;
import java.util.concurrent.LinkedBlockingQueue;

public class Main {
    public static void main(String[] args) {
        Main main = new Main();
        new Thread(main::producer, "producer thread").start();
        new Thread(main::consumer, "consumer thread").start();
        System.out.println("主线程执行到最后一行！");
    }

    //----------------------------------------
    private static final Queue<String> stringList = new LinkedBlockingQueue<>();

    /*
     以stringList为monitor，一次往其中添加一条信息，直到加到五条，然后将所锁对象让给消费者
     */
    public void producer() {
        while (true) {
            System.out.println("生产者 进入 while 循环第一行");
            synchronized (stringList) {

                while (stringList.size() < 5) {
                    String temp = System.currentTimeMillis() + Util.getFiveRandomChar();
                    System.out.println(Thread.currentThread().getName() + " : " + "producer will add: " + temp);
                    stringList.add(temp);

                }


                stringList.notifyAll();//通知其他以stringList为锁的同步块，可以竞争锁资源了

                try {
                    stringList.wait();//加满了，让出monitor，自身处于blocking状态
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println("生产者 同步块 最后一行");
            }

            System.out.println("生产者 while循环 最后一行");
        }
    }

    /*
    消费者，取得锁之后就去消费，打印相应的值并且将值删除
     */
    public void consumer() {
        while (true) {
            System.out.println("消费者 进入 while 循环第一行");
            synchronized (stringList) {

                while (!stringList.isEmpty()) {
                    System.out.println(Thread.currentThread().getName() + " : " + "consumer will remove: " + stringList.peek());
                    stringList.remove();
                }


                stringList.notifyAll();//通知其他的线程可以竞争monitor资源了

                try {
                    stringList.wait();//全部消费完毕，让monitor资源
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                System.out.println("消费者 同步块 最后一行");
            }

            System.out.println("生产者 while循环 最后一行");
        }
    }

    /*
    wait和notify必须在同步块中，并且，必须通过被monitor的对象进行调用！
     */
}
```
执行结果片段：
```shell
（……）
消费者 同步块 最后一行
生产者 while循环 最后一行
消费者 进入 while 循环第一行
consumer thread : consumer will remove: 1564539779546wntlj
consumer thread : consumer will remove: 1564539779546ooncd
consumer thread : consumer will remove: 1564539779546jnyoj
consumer thread : consumer will remove: 1564539779546dtbei
consumer thread : consumer will remove: 1564539779546maawr
生产者 同步块 最后一行
生产者 while循环 最后一行
生产者 进入 while 循环第一行
producer thread : producer will add: 1564539779546zrvby
producer thread : producer will add: 1564539779546ymrob
producer thread : producer will add: 1564539779546ocyoh
producer thread : producer will add: 1564539779546huanv
producer thread : producer will add: 1564539779546nhxda
消费者 同步块 最后一行
生产者 while循环 最后一行
消费者 进入 while 循环第一行
consumer thread : consumer will remove: 1564539779546zrvby
consumer thread : consumer will remove: 1564539779546ymrob
consumer thread : consumer will remove: 1564539779546ocyoh
consumer thread : consumer will remove: 1564539779546huanv
consumer thread : consumer will remove: 1564539779546nhxda
（……）
```
正常了，实现了，最终实现了生产往列表中加5个消息，通知消费者消费，消费完，消费者通知生产者加消息。
# 5 改进
mutex、Lock、Condition什么的可以再探索一下。
