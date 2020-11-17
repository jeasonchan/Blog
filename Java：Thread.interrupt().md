# 1 背景
有个老哥问如果中断一个Java线程，想起来之前看到的一个回答，再次记录、加深一下印象。

参考文章：

Java里一个线程调用了Thread.interrupt()到底意味着什么？ - Intopass的回答 - 知乎
https://www.zhihu.com/question/41048032/answer/89431513

Java里一个线程调用了Thread.interrupt()到底意味着什么？ - 大闲人柴毛毛的回答 - 知乎
https://www.zhihu.com/question/41048032/answer/252905837

# 2 详解
首先，一个线程不应该由其他线程来强制中断或停止，而是应该由线程自己自行停止。所以，Thread.stop, Thread.suspend, Thread.resume 都已经被废弃了。而 Thread.interrupt 的作用其实也不是中断线程，而是「通知线程应该中断了」，具体到底中断还是继续运行，应该由被通知的线程自己处理。

具体来说，当对一个线程，调用 interrupt() 时，① 如果线程处于被阻塞状态（例如处于sleep, wait, join 等状态），那么线程将立即退出被阻塞状态，并抛出一个InterruptedException异常。仅此而已。② 如果线程处于正常活动状态，那么会将该线程的中断标志设置为 true，仅此而已。被设置中断标志的线程将继续正常运行，不受影响。


