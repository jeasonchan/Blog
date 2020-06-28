@[TOC]
# 1配置元数据
Bean是被实例化的、被组装起来的、被IOC容器管理的类实例（一条Bean配置元数据可以产生多个类实例）。创建这些Bean需要的一些配置参数，叫做配置元数据，配置元数据肯定是我们自己写的，之前的提到的提供配置元数据的方法有xml和java类，还有一个就是component注解（一些老教程都没提……搞得我一直以为大家还都在用xml配置元数据……）。配置元数据要包含的具体的参数如下：
* class：这个的值必须有，且指向一个具体存在的java类，bean就是这个java类的实例
* name：bean的唯一标识，且不能以大写字母开头，和前端里的标签的id/name一样
* constructor-arg ：它是用来注入依赖关系的，其实就是构造方法的入参
* properties：它是用来注入依赖关系的，对了类的属性进行赋值，可以是赋值基本数据类型，也可以是赋值一个类实例（在spring中，就是一个由IOC容器控制的bean了）
* autowiring mode：它是用来注入依赖关系的，暂时不太了解
* lazy-initialization mode：延迟初始化的bean，告诉 IOC容器在它第一次被请求时，而不是在启动时去创建一个 bean 实例。debug运行spring demo，发现，不显式配置就默认为，第一次使用bean时创建这个bean，即lazy-initialization=“true”
* initialization 方法：在 bean 的所有必需的属性被容器设置之后，可以简单理解为构造函数之后的，自动执行的方法。
destruction 方法：当包含该 bean 的容器被销毁时，使用该方法，应该是做一些资源释放之类的。
# 2bean和spring容器的关系
下图是bean和spring的简单关系，也能看出bean的生成过程。
![bean和spring容器的关系](https://img-blog.csdnimg.cn/20191006153040634.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2plYXNvbl9jaGFuX3pqdQ==,size_16,color_FFFFFF,t_70)来看一下xml提供的配置元数据实例，配置文件中有不同的 bean 定义，包括延迟初始化，初始化方法和销毁方法的：
```xml
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

   <!-- A simple bean definition -->
   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with lazy init set on -->
   <bean id="..." class="..." lazy-init="true">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with initialization method -->
   <bean id="..." class="..." init-method="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- A bean definition with destruction method -->
   <bean id="..." class="..." destroy-method="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

   <!-- more bean definitions go here -->

</beans>
```

