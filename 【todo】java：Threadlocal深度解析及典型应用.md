# 1 前言
Java并发编程：深入剖析ThreadLocal     https://www.cnblogs.com/dolphin0520/p/3920407.html

要点：

1、每一个线程对象（Thread）实例，都有一个非静态属性，threadLocals（一开始是null的），是一个Map<ThreadLocal,Object>类型的Map

2、ThreadLocal其实是一个数据容器，装在这个容器中的对象在get时都会从threadLocals中取初始值的拷贝使用

3、对ThreadLocal的实例调用set方法，设置希望被使用的初始值，之后在各个线程中get出来的就是最近set进去的对象的拷贝。**set可以理解为：将set的值拷进当前线程的threadLocals中，get可以理解为：当前线程的threadLocals中以ThreadLocal为key取值**


# 2 解析
# 3 代码实践
## 3.1 TransactionTemplate分别设置事务隔离

先看如何获取“独立”的TransactionTemplate对象

```
package com.jeasonchan.dao;

import lombok.Getter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;
import org.springframework.stereotype.Service;
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.support.TransactionTemplate;

import javax.sql.DataSource;

@Service("dBService1")

public class DBService1 {
    @Getter
    private DataSource dataSource;

    @Getter
    private JdbcTemplate jdbcTemplate;

    @Getter
    private DataSourceTransactionManager dataSourceTransactionManager;

    @Getter
    private TransactionTemplate defaultTransactionTemplate;

    @Getter
    private ThreadLocal<TransactionTemplate> transactionTemplateThreadLocal = new ThreadLocal<>();

    public void setIsolatoinLevel(int isolatoinLevel) {
        transactionTemplateThreadLocal.set(new TransactionTemplate(
                this.dataSourceTransactionManager,
                new TransactionDefinition() {
                    @Override
                    public int getIsolationLevel() {
                        return isolatoinLevel;
                    }
                }
        ));
    }

    public TransactionTemplate getCustomTransactionTemplate() {
        if (null == this.transactionTemplateThreadLocal.get()) {
            setIsolatoinLevel(TransactionDefinition.ISOLATION_DEFAULT);
        }
        return this.transactionTemplateThreadLocal.get();
    }


    @Autowired
    public DBService1(@Qualifier("dataSource1") DataSource dataSource) {
        this.dataSource = dataSource;
        this.jdbcTemplate = new JdbcTemplate(dataSource);
        this.dataSourceTransactionManager = new DataSourceTransactionManager(dataSource);
        this.defaultTransactionTemplate = new TransactionTemplate(dataSourceTransactionManager);
    }
}
```

再看一下，实际的调用代码：

```java
System.out.println(System.identityHashCode(dbService1.getCustomTransactionTemplate()));
//这一行获取的是隔离等级为默认的-1的TransactionTemplate实例，只不过只存在于当前线程，并且和defaultTransactionTemplate不是同一个对象

dbService1.setIsolatoinLevel(level);
//经过这次set操作，get出来的内容以发生变化，已经覆盖了Map里的对象

TransactionTemplate customTransactionTemplate = bService1.getCustomTransactionTemplate();

try {
    customTransactionTemplate.execute(new TransactionCallbackWithoutResult() {
        @Override
        protected void doInTransactionWithoutResult(TransactionStatus status) {
            //放事务操作
        }
    });

} catch (TransactionException e) {
    e.printStackTrace();
}
```





## 3.2 i18n

**错误示范**

```java
public class CommonI18n {
    private static Logger logger = LoggerFactory.getLogger(CommonI18n.class);
    
    private final static Map<String, String> zhI18nMap = new HashMap<>();
    private final static Map<String, String> enI18nMap = new HashMap<>();
    
    public static enum LanguageTypeEnum {
        EN("en_US"), ZH("zh_CN");

        private String type;

        LanguageTypeEnum(String languageType) {
            type = languageType;
        }
    }

    //该值决定了使用哪中文Map，一开始初始化为 英文，防止使用者忘记set，直接使用时报错
    private static ThreadLocal<String> languageHolder = new ThreadLocal<String>() {
        {
            //设置ThreadLocal容器的初始值
            //该种方法只能保证在主线程掉哟过有默认值
            //static 是在主线程完成set的初始化的，所以主线程直接get就能得到
            this.set(LanguageTypeEnum.EN.type);
        }
    };
    
    private static CommonI18n instance;

    public static void setLanguageHolder(LanguageTypeEnum languageTypeEnum) {

        languageHolder.set(languageTypeEnum.type);
    }

    public static String getI18nStr(String label) {
        if ("zh_CN".equals(languageHolder.get())) {
            return zhI18nMap.get(label);
        } else {
            return enI18nMap.get(label);
        }
    }


}
```

**正确示范**：

```java
package default_package.ThreadLocal实验;

public class Util2 {
    private static ThreadLocal<String> stringThreadLocal = new ThreadLocal<String>() {
        
        //在子线程里，threadLocal对象没有发生任何set操作，只能返回super.initialValue()，即为null
        //要想改变默认值，重写super.initialValue()即可，见Util2
        
        @Override
        protected String initialValue() {
            return "super.initialValue()";
        }
    };


    public static void setContent(String content) {
        stringThreadLocal.set(content);
    }

    public static String getContent() {
        return stringThreadLocal.get();
    }
}
```


虽然是的静态的方法，但是，每个线程都可以设置自己的目标语言类型，也不必转化为的非静态类，避免了每次都要new一个对象，一定程度上能减少CPU和内存消耗

# 4 重要
一定要在finally中remove掉在当前线程中添加的拷贝值，也就是对ThreadLocal实例进行remove操作，尤其是使用了线程池的情况下，可以避免内存泄露（我们set进线程池线程的变量可能永远不会被访问了，内存泄露了）和变量污染（别人使用你用过的线程池，里
