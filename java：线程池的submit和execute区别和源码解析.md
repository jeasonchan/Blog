# 1背景
在回顾线程池的使用时，发现submit和execute，有很多相似之处，并对其进行了进一步的探索。
先上结论：
1. 线程池中，不需要返回值的pool.execute(runnable)
2. 线程池中，需要返回值的pool.submit(callable)，或者，先进行包装FutureTask<Integer> futureTask = new FutureTask<Integer>(callable);再executor.submit(futureTask);
3. 普通线程中，不需要返回值可以直接使用new Thread(runnable).start()
4. 普通线程中，需要返回值，先进行包装FutureTask<Integer> futureTask = new FutureTask<Integer>(callable);再new Thread(futureTask).start()，结果通过futureTask的属性方法进行查看
5. （**有可能还有其他方法，水平有限，可能暂未列出**）

# 2代码实践
最原始的submit和execute代码实践，submit执行callable接口类，execute执行runnable接口类
```java
package com.xxx.线程池实践;

import java.util.concurrent.*;

public class CallableTask implements Callable, Runnable {
    private int flag;
    public static int ExceptionNumber = 666;

    public CallableTask(int flag) {
        this.flag = flag;
    }

    @Override
    public String call() throws Exception {
        System.out.println(Thread.currentThread().getName());
        if (flag == ExceptionNumber) {
            throw new Exception("flag is ExceptionNumber");
        }
        return flag % 2 == 0 ? "偶数" : "奇数";
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
        System.out.println(flag % 2 == 0 ? "偶数" : "奇数");
    }


    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService cachedThreadPool = Executors.newCachedThreadPool();
        Runnable runnable1 = new CallableTask(1);
        Runnable runnable2 = new CallableTask(2);
        Runnable runnable3 = new CallableTask(666);
        cachedThreadPool.execute(runnable1);
        cachedThreadPool.execute(runnable2);
        cachedThreadPool.execute(runnable3);

        Thread.sleep(1000);

        Callable callable1 = new CallableTask(1);
        Callable callable2 = new CallableTask(2);
        Callable callable3 = new CallableTask(666);
        Future future1 = cachedThreadPool.submit(callable1);
        Future future2 = cachedThreadPool.submit(callable2);
        Future future3 = cachedThreadPool.submit(callable3);
        while (true) {
            if (future1.isDone() && future2.isDone() && future3.isDone()) {
                System.out.println("=====================inside=================while=====================");
                System.out.println(future1.get());//future 的 get 方法本身就是阻塞的，直接调用时会一直等到有了结果才会执行下一条语句
                System.out.println(future2.get());
                try {
                    //get 只能get到call方法的返回值，抛出的异常从调用get方法时一同抛出，异常还是应该使用try catch 捕捉
                    System.out.println(future3.get());
                } catch (Exception e) {
                    System.out.println(e);
                }

//                System.out.println(future3.get());
                break;
            }
            System.out.println("last line in while cycle");
        }
        cachedThreadPool.shutdown();
    }
}

```

#  3 源码解析
可参考这篇大佬的文章：
<https://blog.csdn.net/kai3123919064/article/details/90343380>
