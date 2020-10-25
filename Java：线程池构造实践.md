
```java

import com.google.common.util.concurrent.ThreadFactoryBuilder;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

/**
 * 提供计算机型和IO型线程池
 */
public class ThreadPools {
    private static ExecutorService iOThreadPool;
    private static ExecutorService cPUThreadPool;

    static {
        int numberOfCores = Runtime.getRuntime().availableProcessors();
        double iOBlockingCoefficient = 0.5;
        double cpuBlockingCoefficient = 1;

        int iOPoolSize = (int) (numberOfCores / iOBlockingCoefficient);
        int cPUPoolSize = (int) (numberOfCores / cpuBlockingCoefficient);

        iOThreadPool = new ThreadPoolExecutor(iOPoolSize + 1, iOPoolSize + 1, 0, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>(1024), new ThreadFactoryBuilder().setNameFormat("io-pool-threads-%d").build());

        cPUThreadPool = new ThreadPoolExecutor(cPUPoolSize + 1, cPUPoolSize + 1, 0, TimeUnit.MILLISECONDS,
                new LinkedBlockingQueue<>(1024), new ThreadFactoryBuilder().setNameFormat("cpu-pool-threads-%d").build());


    }

    public static ExecutorService getIOThreadPool() {
        return iOThreadPool;
    }

    public static ExecutorService getCPUThreadPool() {
        return cPUThreadPool;
    }

}
```
