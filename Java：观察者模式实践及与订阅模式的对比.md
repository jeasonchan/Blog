# 1 前言

参考文章：

Java基础类库之Observable   https://blog.csdn.net/qq_18948359/article/details/89893576

观察者模式对比订阅模式   https://zhuanlan.zhihu.com/p/51357583

Java 9：Observer和Observable废弃原因及解决方案    https://majing.io/posts/10000001281162



# 2 JDK实现源码及代码实践
直接看被观察者通知观察者的源码：

```java
    public void notifyObservers(Object arg) {
        /*
         * a temporary array buffer, used as a snapshot of the state of
         * current Observers.
         */
        Object[] arrLocal;

        synchronized (this) {
            /* We don't want the Observer doing callbacks into
             * arbitrary code while holding its own Monitor.
             * The code where we extract each Observable from
             * the Vector and store the state of the Observer
             * needs synchronization, but notifying observers
             * does not (should not).  The worst result of any
             * potential race-condition here is that:
             * 1) a newly-added Observer will miss a
             *   notification in progress
             * 2) a recently unregistered Observer will be
             *   wrongly notified when it doesn't care
             */

            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }
```

可见：

1. 被观察者是否发送通知，取决于变量change；发送通知后，change状态会被自动清除（clearChange）
2. 观察者的update方法其实是被 被观察者 直接调用的，是阻塞式的通知
3. 有一定的线程安全问题，在for循环中发送通知的过程中，下一个变更已经发生，则发送的arg对象引用，可能已经跟一开始的发生变化了，即，**arg始终是最新的，而不是通知发生时的拷贝，高速更新+大量观察者时会有一定的差别**


## 2.1 观察者模式代码实践
被观察者是房子，观察者是购房人，当房价发生变化时，购房者会收到最新的房价信息。代码实现基于类Observable和接口Observer。

被观察者，House类

```java
package com.jeasonchan.dailyexercise.观察者模式实践;

import lombok.Getter;

import java.util.Observable;

public class House extends Observable {
    @Getter
    private String houseName = "House";

    @Getter
    private int price = 0;


    public void setPrice(int newPrice) {
        this.price = newPrice;

        //更改标志，设置已变更状态
        //下一步才能根据已变更的状态发送变量
        super.setChanged();

        //传递想被监控到的变化对象
        super.notifyObservers(newPrice);

    }

    @Override
    public String toString() {
        return "House{" +
                "houseName='" + houseName + '\'' +
                ", price=" + price +
                '}';
    }
}
```

观察者，HouseBuyer

```java
package com.jeasonchan.dailyexercise.观察者模式实践;

import java.util.Observable;
import java.util.Observer;

public class HouseBuyer implements Observer {


    @Override
    public void update(Observable o, Object arg) {


        System.out.println("被观察的对象是：" + o +
                "被观察者通知的内容是：" + arg);



    }
}
```

测试类Controller

```java
package com.jeasonchan.dailyexercise.观察者模式实践;

import java.util.HashMap;
import java.util.Map;

public class ObserverController {

    public static void main(String[] args) {

        HouseBuyer houseBuyer = new HouseBuyer();

        House house = new House();
        
        //将观察者添加进待通知的列表
        house.addObserver(houseBuyer);

        for (int i = 0; i < 10; ++i) {
            System.out.println("设置房价为：" + i);
            
            //一设置新房价，购房者就会收到通知
            house.setPrice(i);
        }


    }


}
```


# 3 和订阅模式进行比较


# 4 Java 9中被废弃的原因及替代方案