@[TOC]
# 线程池
并发线程过多，线程的频繁启动和销毁需要大量时间，会大大降低系统效率，为了使某个线程执行完一个任务后，不被销毁而能够转而继续执行其他任务，使用线程池可以达到这样的效果。

线程池的使用步骤：
1. 创建线程池
2. 创建任务
3. 执行任务
4. 关闭线程池
## 创建线程池
创建线程池主要有两种方式，第一种是使用最原始的线程池构造函数创建自定义的线程池，第二种是直接使用已经配置好的现成构造函数，本质上还是调用第一种自定义的方法， 不过是一些参数不用再设置。

先看第一种通过原始构造方法创建线程池。

线程池的使用主要通过java.uitl.concurrent.ThreadPoolExecutor类实现，此类的构造方法中的主要参数有corePoolSize（核心池线程输数量）、maximumPoolSize（最大线程数量）、KeepAliveTime（空闲线程的存活时间）、workQueue（阻塞任务队列）等等，线程池从任务队列中读取任务并视情况创建线程执行或者复用旧线程执行，核心线程池大小是线程池的一个“期望”大小，同时能在任务较多的情况下不得不在线程池中创建maximumPoolSize数量的线程；当线程池中的线程数量超过期望的大小（即，corePoolSize）时，KeepAliveTime发挥作用，空闲时间大于等于KeepAliveTime的线程开始被销毁，知道线程数量回到期望的数字。

对ThreadPoolExecutor类构造方法的关键参数进行整理、解释：
* corePoolSize：核心池的大小，期望的线程池大小。默认情况下，在创建了线程池后，线程池中的线程数为0，当有任务来之后，就会创建一个线程去执行任务，当线程池中的线程数目达到corePoolSize后，就会把到达的任务放到缓存队列当中；
* maximumPoolSize：
* keepAliveTime：默认情况下，此参数的功能如前文描述；但是，如果调用了allowCoreThreadTimeOut(boolean)方法，在线程池中的线程数不大于corePoolSize时，keepAliveTime参数也会起作用，直到线程池中的线程数为0；
* unit：参数keepAliveTime的时间单位，有7种取值：
```java
TimeUnit.DAYS;               //天
TimeUnit.HOURS;             //小时
TimeUnit.MINUTES;           //分钟
TimeUnit.SECONDS;           //秒
TimeUnit.MILLISECONDS;      //毫秒
TimeUnit.MICROSECONDS;      //微妙
TimeUnit.NANOSECONDS;       //纳秒
```
* workQueue：一个阻塞队列，用来存储等待执行的任务。ArrayBlockingQueue和PriorityBlockingQueue使用较少，一般使用LinkedBlockingQueue和SynchronousQueue。线程池的排队策略与BlockingQueue有关。
* threadFactory：线程工厂，主要用来创建线程；
* handler：表示当拒绝处理任务时的策略，有以下四种取值：
```java
ThreadPoolExecutor.AbortPolicy; //丢弃任务并抛出RejectedExecutionException异常。 
ThreadPoolExecutor.DiscardPolicy; //也是丢弃任务，但是不抛出异常。 
ThreadPoolExecutor.DiscardOldestPolicy; //丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
ThreadPoolExecutor.CallerRunsPolicy; //由调用线程处理该任务 
```
实际使用代码示例：
```java
ThreadPoolExecutor()创建线程池示例
```


再看第二种，常用的几种模板线程池。

目前来看，有五种线程池模板：
* newSingleThreadExecutor：单例线程，表示在任意的时间段内，线程池中只有一个线程在工作。
* newCachedThreadPool：缓存线程池，一个可以无限扩大的线程池，比较适合处理执行时间比较小的任务。先查看线程池中是否有当前执行线程的缓存，如果有就resue(复用)，如果没有，那么需要创建一个线程来完成当前的调用。keepAliveTime为60秒。
* newFixedThreadPool：固定型线程池，和newCacheThreadPool()差不多，也能够实现resue(复用)，但是这个池子规定了线程的最大数量，也就是说当池子有空闲时，那么新的任务将会在空闲线程中被执行，一旦线程池内的线程都在进行工作，那么新的任务就必须等待线程池有空闲的时候才能够进入线程池，其他的任务继续排队等待，这类池子没有规定其空闲的时间到底有多长.这一类的池子更适用于**服务器**。
* newScheduledThreadPool：调度型线程池，调度型线程池会根据Scheduled(任务列表)进行延迟执行、周期性的执行。延迟还是周期都是根据scheduled来的。
* newWorkStealingPool：一个拥有多个任务队列的线程池，可以减少连接数，创建当前可用cpu数量的线程来并行执行。

使用线程池模板创建线程池：
```java
ExecutorService executorService1=Executors.newSingleThreadExecutor();//注意变量名，已经包含一个new了
ExecutorService executorService2=Executors.newSingleThreadScheduledExecutor();
ExecutorService executorService3=Executors.newFixedThreadPool(10);//适合服务器
ExecutorService executorService4=Executors.newScheduledThreadPool(10);
```
## 创建任务
任务分为两种，实现了Callable接口或者实现了Runnable接的类。
* 无返回值的任务就是一个实现了runnable接口的类，覆写run()方法实现线程业务逻辑，run()方法不能有返回值，不能抛出异常。
* 有返回值的任务是一个实现了callable接口的类，覆写call()方法实现线程业务逻辑，call()方法可以有返回值，可以抛出异常。
此处使用Callable和Future实现创建任务：
```java
class CallableAndFuture implements Callable{
    private int flag=0;

    public CallableAndFuture(int flag){
        this.flag=flag;
    }

    @Override
    public String call() throws Exception {
        if(this.flag==0){
            return "flag=0";
        }

        if(this.flag==1){
            try{
                while (true){
                    System.out.println("Looping!");
                    Thread.sleep(1000);
                }

            }catch (Exception e){
                System.out.println(e.toString());
            }finally {
                return "false";
            }
        }else{//flag不为0和1的情况都是直接抛出异常，没有返回值
            throw new Exception("Bad flag number!");
        }

    }

    /**
     * 执行方法，调用此方法时，创建含有三个线程的线程池进行相关处理并将结果进行打印。
     * 其中future类比较神奇，可以详细研究一下源码！
     *
      * @return
     */
    public static Object doit(){
        //声明并定义三个callable类型的任务
        CallableAndFuture taskCallableAndFuture0=new CallableAndFuture(0);
        CallableAndFuture taskCallableAndFuture1=new CallableAndFuture(1);
        CallableAndFuture taskCallableAndFuture2=new CallableAndFuture(2);

        //创建一个执行任务的线程池
        ExecutorService executorService=Executors.newFixedThreadPool(3);

        try{
            //提交并执行任务，任务启动时返回一个future类实例
            //如果想得到任务执行的结果 或者 异常，可对这个future类实例记性操作
            Future future0=executorService.submit(taskCallableAndFuture0);

            //使用future类的get()方法时，当前线程会被阻塞，直到get到执行的结果，是个阻塞方法
            System.out.println("taskCallableAndFuture0 执行结果是:"+future0.get());

            Future future1=executorService.submit(taskCallableAndFuture1);
            Thread.sleep(5000);
            System.out.println("taskCallableAndFuture1:"+future1.cancel(true));

            //获取第三个任务的输出，因为第三个任务注定会抛出异常，这里只将异常进行打印，异常打印也是一种异常处理方式
            Future future2=executorService.submit(taskCallableAndFuture2);
            System.out.println("taskCallableAndFuture2 执行结果是:"+future2.get());

            //运行到此处，应该是有四个线程：线程池三个，外加主线程，且主线程中的get在等着线程池线程执行完毕
        }catch (Exception e){
            System.out.println(e.toString());
        }finally {
            executorService.shutdownNow();
        }
        return null;
    }
}
```
## 执行任务
ExecutorService executorService=Executors.newFixedThreadPool(3);
通过java.util.concurrent.ExecutorService接口对象来执行任务，该对象有两个方法可以执行任务execute和submit。execute这种方式提交没有返回值，也就不能判断是否执行成功。submit这种方式它会返回一个Future对象，通过future的get方法来获取返回值，get方法会阻塞住直到任务完成。
execute与submit区别：

接收的参数不一样
submit有返回值，而execute没有
submit方便Exception处理
execute是Executor接口中唯一定义的方法；submit是ExecutorService（该接口继承Executor）中定义的方法
## 关闭线程池
线程池使用完毕，需要对其进行关闭，有两种方法：

shutdown()，不再接受新的任务，如果线程池内有任务，那么把这些任务执行完毕后，关闭线程池。

shutdownNow()，不再接受新的任务，并把任务队列中的任务直接移出掉，如果有正在执行的，尝试进行停止。
## 综合使用案例（Future Task）

```java
import java.util.concurrent.*;
 
/**
 * Author  : Slogen
 * AddTime : 17/6/4
 * Email   : huangjian13@meituan.com
 */
public class CallDemo {
 
    public static void main(String[] args) throws ExecutionException, InterruptedException {
 
        /**
         * 第一种方式:Future + ExecutorService
         * Task task = new Task();
         * ExecutorService service = Executors.newCachedThreadPool();
         * Future<Integer> future = service.submit(task1);
         * service.shutdown();
         */
 
 
        /**
         * 第二种方式: FutureTask + ExecutorService
         * ExecutorService executor = Executors.newCachedThreadPool();
         * Task task = new Task();
         * FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
         * executor.submit(futureTask);
         * executor.shutdown();
         */
 
        /**
         * 第三种方式:FutureTask + Thread
         */
 
        // 2. 新建FutureTask,需要一个实现了Callable接口的类的实例作为构造函数参数
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Task());
        // 3. 新建Thread对象并启动
        Thread thread = new Thread(futureTask);
        thread.setName("Task thread");
        thread.start();
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
 
        // 4. 调用isDone()判断任务是否结束
        if(!futureTask.isDone()) {
            System.out.println("Task is not done");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        int result = 0;
        try {
            // 5. 调用get()方法获取任务结果,如果任务没有执行完成则阻塞等待
            result = futureTask.get();
        } catch (Exception e) {
            e.printStackTrace();
        }
 
        System.out.println("result is " + result);
 
    }
 
    // 1. 继承Callable接口,实现call()方法,泛型参数为要返回的类型
    static class Task  implements Callable<Integer> {
 
        @Override
        public Integer call() throws Exception {
            System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
            int result = 0;
            for(int i = 0; i < 100;++i) {
                result += i;
            }
 
            Thread.sleep(3000);
            return result;
        }
    }
}
```
## 综合使用案例一
需求：从数据库中获取url，并利用httpclient循环访问url地址，并对返回结果进行操作

分析：由于是循环的对多个url进行访问并获取数据，为了执行的效率，考虑使用多线程，url数量未知如果每个任务都创建一个线程将消耗大量的系统资源，最后决定使用线程池。
```java
public class GetMonitorDataService {
 
    private Logger logger = LoggerFactory.getLogger(GetMonitorDataService.class);
    @Resource
    private MonitorProjectUrlMapper groupUrlMapper;
    @Resource
    private MonitorDetailBatchInsertMapper monitorDetailBatchInsertMapper;
    public void sendData(){
        //调用dao查询所有url
        MonitorProjectUrlExample example=new MonitorProjectUrlExample();
        List<MonitorProjectUrl> list=groupUrlMapper.selectByExample(example);
        logger.info("此次查询数据库中监控url个数为"+list.size());
 
        //获取系统处理器个数，作为线程池数量
        int nThreads=Runtime.getRuntime().availableProcessors();
 
        //定义一个装载多线程返回值的集合
        List<MonitorDetail> result= Collections.synchronizedList(new ArrayList<MonitorDetail>());
        //创建线程池，这里定义了一个创建线程池的工具类，避免了创建多个线程池，ThreadPoolFactoryUtil可以使用单例模式设计
        ExecutorService executorService = ThreadPoolFactoryUtil.getExecutorService(nThreads);
        //遍历数据库取出的url
        if(list!=null&&list.size()>0) {
            for (MonitorProjectUrl monitorProjectUrl : list) {
                String url = monitorProjectUrl.getMonitorUrl();
                //创建任务
                ThreadTask threadTask = new ThreadTask(url, result);
                //执行任务
                executorService.execute(threadTask);
               
                try {//等待直到所有任务完成
                          executorService.awaitTermination(Long.MAX_VALUE, TimeUnit.MINUTES);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
 //注意区分shutdownNow
            executorService.shutdown();
            //对数据进行操作
            saveData(result);
        }
    }

```
任务
```java
public class ThreadTask implements Runnable{
    //这里实现runnable接口
    private String url;
    private List<MonitorDetail> list;
    public ThreadTask(String url,List<MonitorDetail> list){
        this.url=url;
        this.list=list;
    }
    //把获取的数据进行处理
    @Override
    public void run() {
        MonitorDetail detail = HttpClientUtil.send(url, MonitorDetail.class);
        list.add(detail);
    }

}
```
## 综合使用案例二（countDownLatch）
```java
package com.br.lucky.utils;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * @author 10400
 * @create 2018-04-19 20:38
 */
public class FatureTest {

    //1、配置线程池
    private static ExecutorService es = Executors.newFixedThreadPool(20);

    //2、封装响应Feature
    class BizResult{
        public String orderId;
        public String  data;

        public String getOrderId() {
            return orderId;
        }

        public void setOrderId(String orderId) {
            this.orderId = orderId;
        }

        public String getData() {
            return data;
        }

        public void setData(String data) {
            this.data = data;
        }
    }


    //3、实现Callable接口
    class BizTask implements Callable {

        private String orderId;

        private Object data;

        //可以用其他方式
        private CountDownLatch countDownLatch;

        public BizTask(String orderId, Object data, CountDownLatch countDownLatch) {
            this.orderId = orderId;
            this.data = data;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public Object call() {
            try {
                //todo business
                System.out.println("当前线程Id = " + this.orderId);
                BizResult br = new BizResult();
                br.setOrderId(this.orderId);
                br.setData("some key about your business" + this.getClass());
                return br;
            }catch (Exception e){
                e.printStackTrace();
            }finally {
                //线程结束时，将计时器减一
                countDownLatch.countDown();
            }
            return null;
        }
    }

    /**
     * 业务逻辑入口
     */
    public List<Future> beginBusiness() throws InterruptedException {
        //模拟批量业务数据
        List<String> list = new ArrayList<>();
        for (int i = 0 ; i < 1000 ; i++) {
            list.add(String.valueOf(i));
        }
        //设置计数器
        CountDownLatch countDownLatch = new CountDownLatch(list.size());

        //接收多线程响应结果
        List<Future> resultList = new ArrayList<>();
        //begin thread
        for( int i = 0 ,size = list.size() ; i<size; i++){
            //todo something befor thread
            resultList.add(es.submit(new BizTask(list.get(i), null, countDownLatch)));
        }
        //wait finish
        countDownLatch.await();
        return resultList;
    }

    public static void main(String[] args) throws InterruptedException {
        FatureTest ft = new FatureTest();
            List<Future> futures = ft.beginBusiness();
            System.out.println("futures.size() = " + futures.size());
            //todo some operate
            System.out.println(" ==========================end========================= " );
    }

}


```

## 综合使用案例三（future.get()）
```java
package com.br.lucky.utils;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

/**
 * @author 10400
 * @create 2018-04-19 20:38
 */
public class FatureTest {

/**
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
                .setNameFormat("demo-pool-%d").build();
        ExecutorService pool = new ThreadPoolExecutor(5, 200,0L, TimeUnit.MILLISECONDS,new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.CallerRunsPolicy());
        for(int i=0;i<1000;i++) {
            pool.execute(() -> {
                //todo 业务逻辑在此
            });
        }
*/
    //1、配置线程池
    private static ExecutorService es = Executors.newFixedThreadPool(20);

    //2、封装响应Feature
    class BizResult{
        public String orderId;
        public String  data;

        public String getOrderId() {
            return orderId;
        }

        public void setOrderId(String orderId) {
            this.orderId = orderId;
        }

        public String getData() {
            return data;
        }

        public void setData(String data) {
            this.data = data;
        }
    }


    //3、实现Callable接口
    class BizTask implements Callable {

        private String orderId;

        private Object data;


        public BizTask(String orderId, Object data) {
            this.orderId = orderId;
            this.data = data;
        }

        @Override
        public Object call() {
            try {
                //todo business
                System.out.println("当前线程Id = " + this.orderId);
                BizResult br = new BizResult();
                br.setOrderId(this.orderId);
                br.setData("some key about your business" + this.getClass());
                Thread.sleep(3000);
                return br;
            }catch (Exception e){
                e.printStackTrace();
            }
            return null;
        }
    }

    /**
     * 业务逻辑入口
     */
    public List<Future> beginBusiness() throws InterruptedException, ExecutionException {
        //模拟批量业务数据
        List<String> list = new ArrayList<>();
        for (int i = 0 ; i < 100 ; i++) {
            list.add(String.valueOf(i));
        }

        //接收多线程响应结果
        List<Future> resultList = new ArrayList<>();
        //begin thread
        for( int i = 0 ,size = list.size() ; i<size; i++){
            //todo something befor thread
            Future future = es.submit(new BizTask(list.get(i), null));
            resultList.add(future);
        }

        for (Future f : resultList) {
            f.get();
        }

        System.out.println(" =====多线程执行结束====== ");

        //wait finish
        return resultList;
    }

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        FatureTest ft = new FatureTest();
            List<Future> futures = ft.beginBusiness();
            System.out.println("futures.size() = " + futures.size());
            //todo some operate
            System.out.println(" ==========================end========================= " );
    }

}

```
参考文章：
<https://www.jianshu.com/p/edd7cb4eafa0>
<https://yq.aliyun.com/articles/5952>
<http://www.importnew.com/25286.html>
