@[TOC]
# 1 概览
两种方式进行beans的装配：xml和java类
初始化使用过的applicationContext实现分别是：ClassPathXmlApplicationContext和AnnotationConfigApplicationContext
# 2 代码实践
**Person.java**
```java
package org.jeasonchan;

public interface Peson {
    public void sayHelloAndName();
}
```
接口的两个实现类，**Chinese.java**和**American.java**，就是实现了sayHello()方法，打印各自属性“name”。
```java
package org.jeasonchan;

import lombok.Getter;
import lombok.Setter;

public class Chinese implements Peson {
    @Setter
    @Getter
    private String name;

    @Override
    public void sayHelloAndName() {
        System.out.println("Hello, I am "+this.name);
    }
}
```
```java
package org.jeasonchan;

import lombok.Getter;
import lombok.Setter;

public class American implements Peson {
    @Setter
    @Getter
    private String name;

    @Override
    public void sayHelloAndName() {
        System.out.println("Hello, I am "+this.name);
    }
}
```
## 2.1 使用xml方式进行beans装配
在resources文件夹下面新建xml配置文件，文件名随意，以我为例，**beansConfig.xml**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="chinese" class="org.jeasonchan.Chinese">
        <property name="name" value="Geninus"/>
    </bean>

    <bean id="american" class="org.jeasonchan.American">
        <property name="name" value="Bitch"/>
    </bean>

</beans>
```
注意！！！bean的Id一定是消息字母开头，spring是为了防止和类的构造方法重名。
装配完毕，开始在主方法里进行调用，分别产生Chinese和American的实例，并调用实例方法的sayHello()方法。
**AppBasedXmlConfig .java**
```java
package org.jeasonchan;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Hello world!
 */
public class AppBasedXmlConfig {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beansConfig.xml");
        Chinese chinese = (Chinese) context.getBean("chinese");
        chinese.sayHelloAndName();
        American american = (American) context.getBean("american");
        american.sayHelloAndName();
    }
}
```
右键main函数运行，收工。
## 2.2使用java类方式进行beans装配
就是使用一个类来代替xml文件，以我为例：
**BeansJavaConfig.java**
```java
package org.jeasonchan;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class BeansJavaConfig {
    @Bean
    public Chinese chinese(){
        Chinese result=new Chinese();
        result.setName("Genius");
        return result;
    }

    @Bean
    public American american(){
        American result=new American();
        result.setName("Bitch");
        return result;
    }
}
```
装配完毕，开始在主方法里进行调用，分别产生Chinese和American的实例，并调用实例方法的sayHello()方法。
**AppBasedJavaConfig .java**
```java
package org.jeasonchan;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class AppBasedJavaConfig {
    public static void main(String[] args) {
        ApplicationContext context=new AnnotationConfigApplicationContext(BeansJavaConfig.class);
        Chinese chinese = (Chinese) context.getBean("chinese");
        chinese.sayHelloAndName();
        American american = (American) context.getBean("american");
        american.sayHelloAndName();
    }

}
```
# 3 总结
可见，目前看见的项目一般都是使用xml文件作为装配配置文件，可能看上去比较简洁，并且可以明显看出，java类的装配方式其实就是xml换个表达方式。
