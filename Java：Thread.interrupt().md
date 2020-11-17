# 1 背景
有个老哥问如果中断一个Java线程，想起来之前看到的一个回答，再次记录、加深一下印象。

参考文章：

Java里一个线程调用了Thread.interrupt()到底意味着什么？ - Intopass的回答 - 知乎
https://www.zhihu.com/question/41048032/answer/89431513

Java里一个线程调用了Thread.interrupt()到底意味着什么？ - 大闲人柴毛毛的回答 - 知乎
https://www.zhihu.com/question/41048032/answer/252905837

# 2 详解
首先，一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止。所以，Thread.stop, Thread.suspend, Thread.resume 都已经被废弃了。而 Thread.interrupt 的作用其实也不是中断线程，而是**通知线程应该中断了**，具体到底中断还是继续运行，应该由被通知的线程自己检查中断状态、自己处理终端状态为true时的操作。

具体来说，当对一个线程，调用 interrupt() 时：
1. 如果线程处于被阻塞状态（例如处于sleep, wait, join 等状态），那么线程将立即退出被阻塞状态，并抛出一个InterruptedException异常。仅此而已。
2. 如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true，仅此而已。被设置中断标志的线程将继续正常运行，不受影响（除非自己的业务里显显式检查中断状态并处理）。

interrupt() 并不能真正的中断线程，需要被调用的线程自己进行配合才行。

也就是说，一个线程如果有被中断的需求，那么就可以这样做：
1. 在正常运行任务时，经常检查本线程的中断标志位，如果被设置了中断标志就自行停止线程。
2. 在调用阻塞方法时正确处理InterruptedException异常。（例如，catch异常后就结束线程。）

比如：

```java
Thread thread = new Thread(() -> {
    while (!Thread.interrupted()) {
        // do more work.
    }
    
    //在此处抛出一个InterruptedException异常
});
thread.start();

// 一段时间以后
thread.interrupt();
```
Thread.interrupted()的作用有两个：
1. 返回当前的中断状态
2. 清除中断状态，也就是将中断状态设为false

清除标志位是为了下次继续检测标志位。如果一个线程被设置中断标志后，选择结束线程那么自然不存在下次的问题，而如果一个线程被设置中断标识后，进行了一些处理后选择继续进行任务，而且这个任务也是需要被中断的，那么当然需要清除标志位了。

注意：
* Thread.interrupted(); 会清除中断状态标志
* Thread.currentThread().isInterrupted();  不会清除中断状态标志
