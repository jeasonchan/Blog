@[TOC]
# 1背景
刚入门spring，很好奇从ioc容器中通过bena的id得到的实例对象，是不是每个都是一样的，就做了试验，发现，默认情况下，自己没有显式配置的情况爱，确实get到同一个对象实例；后来就思考如何从同一个的id的bean中每次都得到同一个实例，结论就是：
```xml
    <bean id="multiChinese" class="Chinese" scope="prototype">
```

其中的scope属性，不定义的时候，默认是singleton的，配置成prototype就是多实例的了。在下文开始验证
# 2代码验证scope="prototype"配置多实例
**beansConfig.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--默认是单例的实例-->
    <bean id="chinese" class="Chinese">
        <property name="name" value="jeasonChan"/>
    </bean>

    <!--多实例的bean-->
    <bean id="multiChinese" class="Chinese" scope="prototype">
        <property name="name" value="multiChinses"/>
    </bean>

</beans>
```
主类，**AppStartWithXmlConfig.java**
```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AppStartWithXmlConfig {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beansConfig.xml");
        Chinese chinese1 = (Chinese) context.getBean("chinese");
        Chinese chinese2 = (Chinese) context.getBean("chinese");
        chinese1.sayHello();
        chinese2.sayHello();
        System.out.println(chinese1);
        System.out.println(chinese2);

        System.out.println("==============================");

        Chinese chinese3 = (Chinese) context.getBean("multiChinese");
        Chinese chinese4 = (Chinese) context.getBean("multiChinese");
        System.out.println(chinese3);
        System.out.println(chinese4);

        /*
        控制台输出：
        jeasonChan
        jeasonChan
        Chinese@6631f5ca
        Chinese@6631f5ca
        ==============================
        Chinese@27ff5d15
        Chinese@4e096385

        可见，默认情况下，同一个的id，从容器里得到的都是同一个实例对象，即，默认是单例的，
        想要多实例，需要改变属性
         */
    }
}
```
从控制台输出来看，不显式配置bean，默认为单例的bean，通过scope="prototype"可实现多实例。
**但是，既然IOC容器能控制单实例了，spring框架的应用，java的单例模式就没有意义了？**
