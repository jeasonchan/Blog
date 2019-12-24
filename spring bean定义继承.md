@[TOC]
# 1什么是定义继承
配置元数据的继承！！！并不是在bean中定义类实例之间的继承工程。配置元数据包含的信息有很多，比如：构造函数的参数、属性的值、bean的作用域信息、初始化和销毁方法、后置处理方法等。
bean定义的继承，就是包括以上配置元数据包含的详细的各种参数的继承。
当我们使用XML作为bean的配置方式时，通过使用父属性，指定父 bean 作为该属性的值来表明子 bean 的定义。
# 2 基于XML的配置元数据继承
**HelloWorld.java**
```java
package com.tutorialspoint;
public class HelloWorld {
   private String message1;
   private String message2;
   public void setMessage1(String message){
      this.message1  = message;
   }
   public void setMessage2(String message){
      this.message2  = message;
   }
   public void getMessage1(){
      System.out.println("World Message1 : " + message1);
   }
   public void getMessage2(){
      System.out.println("World Message2 : " + message2);
   }
}
```
**HelloIndia.java**
```java
package com.tutorialspoint;

public class HelloIndia {
   private String message1;
   private String message2;
   private String message3;

   public void setMessage1(String message){
      this.message1  = message;
   }

   public void setMessage2(String message){
      this.message2  = message;
   }

   public void setMessage3(String message){
      this.message3  = message;
   }

   public void getMessage1(){
      System.out.println("India Message1 : " + message1);
   }

   public void getMessage2(){
      System.out.println("India Message2 : " + message2);
   }

   public void getMessage3(){
      System.out.println("India Message3 : " + message3);
   }
}
```

下面是配置文件 **Beans.xml**，在该配置文件中我们定义有两个属性 message1 和 message2 的 “helloWorld” bean。然后，使用 parent 属性把 “helloIndia” bean 定义为 “helloWorld” bean 的孩子。这个子 bean 继承 message2 的属性，重写 message1 的属性，并且引入一个属性 message3。
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="helloWorld" class="com.tutorialspoint.HelloWorld">
      <property name="message1" value="Hello World!"/>
      <property name="message2" value="Hello Second World!"/>
   </bean>

   <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="helloWorld">
      <property name="message1" value="Hello India!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

</beans>
```

# 3 bean定义的模板
创建一个 Bean 定义模板，不需要花太多功夫它就可以被其他子 bean 定义使用。在定义一个 Bean 定义模板时，你**不应该指定类的属性（不指定class=xxxxx），同时指定带 true 值的抽象属性**，如下所示：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <bean id="beanTeamplate" abstract="true">
      <property name="message1" value="Hello World!"/>
      <property name="message2" value="Hello Second World!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

   <bean id="helloIndia" class="com.tutorialspoint.HelloIndia" parent="beanTeamplate">
      <property name="message1" value="Hello India!"/>
      <property name="message3" value="Namaste India!"/>
   </bean>

</beans>
```
