@[TOC]
# 1bean的生命周期
Bean的生命周期可以表达为：
1. Bean的定义，给出配置元数据，可通过xml、java类、注解三种方式
2. Bean的初始化，调用InitializingBean 接口定义的afterPropertiesSet()方法或者xml中指定的init-method的值
3. Bean的使用，get出来并使用
4. Bean的销毁，调用DisposableBean 接口定义的destroy()方法或者xml中指定的destroy-method的值

当一个 bean 被实例化时，它可能需要执行一些初始化使它转换成可用状态。同样，当 bean 不再需要，并且从容器中移除时，可能需要做一些清除工作。
# 2初始化回调
**接口继承指定初始化回调**
org.springframework.beans.factory.InitializingBean 接口指定一个单一的方法：
```java
public class ExampleBean implements InitializingBean {
   public void afterPropertiesSet() {
      // do some initialization work
   }
}
```
**XML配置元数据指定无返回值且无参数的方法作为初始化回调方法**
java类：
```java
public class ExampleBean {
   public void init() {
      // do some initialization work
   }
}
```
XML的Bean配置文件：
```xml
<bean id="exampleBean" class="examples.ExampleBean" init-method="init"/>
```
# 3销毁回调
**接口继承指定初始化回调**
org.springframework.beans.factory.DisposableBean 接口指定一个单一的方法：
```java
public class ExampleBean implements DisposableBean {
   public void destroy() {
      // do some destroy work
   }
}
```
**XML配置元数据指定无返回值且无参数的方法作为销毁回调方法**
java类：
```java
public class ExampleBean {
   public void destroy() {
      // do some destroywork
   }
}
```
XML的Bean配置文件：
```xml
<bean id="exampleBean" class="examples.ExampleBean" destroy-method="destroy"/>
```
# 4应用场景
如果在**非 web 应用程序环境中使用 Spring 的 IoC 容器**，比如在客户端桌面环境中，那么在 JVM 中我们要注册关闭 hook，这样可以确保正常关闭，让所有的资源都被释放（典型的，数据库连接池的关闭，执行线程池的关闭），可以在单个 beans 上调用 destroy 方法去进行资源释放。不建议使用接口InitializingBean 或者 DisposableBean 的实现回调方法，因为 XML 配置在命名方法上提供了极大的灵活性。
## 4.1批量设置初始化和销毁回调方法
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-3.0.xsd"
    default-init-method="init" 
    default-destroy-method="destroy">
    <--注意！默认初始化和默认销毁回调方法定义的位置！-->

   <bean id="..." class="...">
       <!-- collaborators and configuration for this bean go here -->
   </bean>

</beans>
```
