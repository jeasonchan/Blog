# 1 前言
Java并发编程：深入剖析ThreadLocal     https://www.cnblogs.com/dolphin0520/p/3920407.html

要点：

1、每一个线程对象（Thread）实例，都有一个非静态属性，threadLocals（一开始是null的），是一个Map<ThreadLocal,Object>类型的Map

2、ThreadLocal其实是一个数据容器，装在这个容器中的对象在get时都会从threadLocals中取初始值的拷贝使用

3、对ThreadLocal的实例调用set方法，设置希望被使用的初始值，之后在各个线程中get出来的就是最近set进去的对象的拷贝。**set可以理解为：将set的值拷进当前线程的threadLocals中，get可以理解为：当前线程的threadLocals中以ThreadLocal为key取值**


# 2 解析
# 3 代码实践
## 3.1 数据库连接

## 3.2 i18n
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

虽然是的静态的方法，但是，每个线程都可以设置自己的目标语言类型，也不必转化为的非静态类，避免了每次都要new一个对象，一定程度上能减少CPU和内存消耗

# 4 重要
一定要在finally中remove掉在当前线程中添加的拷贝值，也就是对ThreadLocal实例进行remove操作，尤其是使用了线程池的情况下，可以避免内存泄露（我们set进线程池线程的变量可能永远不会被访问了，内存泄露了）和变量污染（别人使用你用过的线程池，里面还有我们剩下的变量）。
