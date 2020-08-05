# 1 背景
JMeter测试MongoDB  https://www.cnblogs.com/yangxia-test/p/4023905.html

使用 JMeter压测工具 对 MySQL、MongoDB、Neo4j 进行性能测试     https://www.cnblogs.com/zhongyuanzhao000/p/12152843.html

Jmeter让压测随时做起来  https://www.jianshu.com/p/3cc4dd32a89a


# 2 MongoDB压测实践
新版本不官方支持MongoDB的压测了，直接自己写Java代码压测：

```java
package default_package.mongo_test;

import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;

import java.util.Date;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


public class Main {
    private static String IP = "";

    public static void main(String[] args) {
        MongoClient mongoClient = null;
        try {

            mongoClient = MongoClients.create("mongodb://jeason:jeason@xx.xx.xx.xx:27018,xx.xx.xx.xx:27017");

            MongoDatabase myDB = mongoClient.getDatabase("myDB");

            MongoCollection<Document> myCollection = myDB.getCollection("myCollection");

            //写入的线程数，每个线程的写入数量
            int threadNumber = 8;
            int numberForEachThread = 1200;


            CountDownLatch countDownLatch = new CountDownLatch(threadNumber);

            ExecutorService executorService = Executors.newFixedThreadPool(threadNumber);

            System.out.println("开始时间：" + new Date());


            for (int i = 0; i < threadNumber; ++i) {
                executorService.submit(
                        new MongoInsert(countDownLatch, numberForEachThread, "thread_" + i, myCollection)
                );
            }


            countDownLatch.await();
            System.out.println("结束时间：" + new Date());


        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (null != mongoClient) {
                mongoClient.close();
            }
        }


    }


    static class MongoInsert implements Runnable {
        private final CountDownLatch countDownLatch;
        private final int numbers;
        private final String threadName;
        private final MongoCollection<Document> collection;

        public MongoInsert(CountDownLatch countDownLatch, int numbers, String threadName, MongoCollection<Document> collection) {
            this.countDownLatch = countDownLatch;
            this.numbers = numbers;
            this.threadName = threadName;
            this.collection = collection;
        }

        @Override
        public void run() {

            try {
                for (int i = 0; i < this.numbers; ++i) {
                    Document document = new Document();
                    document.append("threadName", this.threadName);
                    document.append("order", i);
                    this.collection.insertOne(document);
                }

            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                this.countDownLatch.countDown();
            }


        }
    }
}

```
