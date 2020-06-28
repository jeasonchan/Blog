@[TOC]
# 1 含义
面向切面编程，原始的定义是，允许我们将遍布应用各处的功能分离出来，形成可重复使用的组件。
## 1.1 为什么要面向切面编程AOP？
1. 首先，日志、服务管理、事务等这样的系统模块，绝对是每个业务代码都需要经常使用的，对这样的系统模块的调用代码肯定在你的代码中随处可见，并且，如果系统模块的调用接口改了，我们的就必须找到每个调用的地方，进行相应的修改，调用点十分多时，就十分痛苦了。
2. 其次，对这些系统模块的调用，根本是业务模块不关心的东西，对系统模块的显式调用也会让代码显得凌乱。
针对这样的需要大量共享、被调用的模块/服务，使用AOP能使这些服务模块化，以声明的方式将这些服务应用到它们需要影响的业务组件/模块中，开发业务组件的人员不需要了解、关注的系统服务的复杂性。最终，AOP（面向切面编程）使业务相关的POJO保持简单。
## 1.2 什么是AOP（面向切面编程）？
上面提到，AOP其实是将日志、安全、事务等业务无关，但是又十分普遍的模块，从业务模块中抽离出来，形象的理解就是，将切面的看成覆盖在业务模块上的外壳，这些外外壳包裹住了核心业务层，这些外壳以声明的方式应用到业务模块中。
接下来从代码实践中看一下AOP（面向切面编程）。
#  2 AOP（面向切面编程）代码实践
实际业务过程中，经常有记录日志记录的需求，以日志记录服务为例，展现没有面向切面编程之前和之后的对比。
原始代码：**突然领悟：spring框架就要求不要出现new这个关键字……一切都是XML文件、java配置类、注解的形式配置的**
```java
package org.springinaction.oldKnights;

public interface DoSth {
    void doSomething();
}
```
```java
package org.springinaction.oldKnights;

import java.io.PrintStream;

public class LogService {
    private PrintStream stream;

    public LogService(PrintStream stream) {
        this.stream = stream;
    }

    public void logBeforeOperatoin() {
        stream.print("log before operation!");
    }

    public void logAfterOperatoin() {
        stream.print("log after operation!");
    }


}
```
```java
package org.springinaction.oldKnights;

public class Person {
    private DoSth doSth;
    private LogService logService;

    public Person(DoSth doSth, LogService logService) {
        this.doSth = doSth;
        this.logService = logService;
    }

    public void triggleDoSthImp() {
        logService.logBeforeOperatoin();
        doSth.doSomething();
        logService.logAfterOperatoin();
    }
}
```
```java
package org.springinaction.oldKnights;

import java.io.PrintStream;

public class Sleep implements DoSth {
    private PrintStream stream;

    public Sleep(PrintStream stream) {
        this.stream = stream;
    }

    @Override
    public void doSomething() {
        stream.print("Sleep!");
    }
}
```
**装配文件如下：**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="sleep" class="org.springinaction.oldKnights.Sleep">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <bean id="logService" class="org.springinaction.oldKnights.LogService">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <bean class="org.springinaction.oldKnights.Person" id="person">
        <constructor-arg index="0" ref="sleep"/>
        <constructor-arg index="1" ref="logService"/>
    </bean>


</beans>

```
**主类文件如下：**
```java
package org.springinaction.oldKnights;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("org/springinaction/oldKnights/emmmm.xml");
        Person person = (Person) context.getBean("person");
        person.triggleDoSthImp();
    }
}
```

可以看出日志服务类在代码层面侵入了业务类Person类，并且在Person编码时，负责该类的程序员还需要关心日志服务类的相关细节，使代码变得复杂，尤其是，以后如果要进行空指针检查的话，代码会更加复杂。
现在，开始将LogService配置为面向切面编程，将其抽象为一个切面。
首先，肯定是去除Person类中对日志服务类的显式使用，得到干净的Person类：
```java
package org.springinaction.newKnights;

import org.springinaction.oldKnights.DoSth;
import org.springinaction.oldKnights.LogService;

public class Person {
    private DoSth doSth;

    public Person(DoSth doSth) {
        this.doSth = doSth;
    }

    public void triggleDoSthImp() {
        doSth.doSomething();
    }
}

```
然后，在对装配文件中对person的装配参数进行修改；然后，增加aop的配置节点，得到新的xml装配文件：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
    <bean id="sleep" class="org.springinaction.oldKnights.Sleep">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <bean id="logService" class="org.springinaction.oldKnights.LogService">
        <constructor-arg value="#{T(System).out}"/>
    </bean>

    <bean class="org.springinaction.newKnights.Person" id="person">
        <constructor-arg index="0" ref="sleep"/>
    </bean>



    <aop:config>
        <!--申明切面，将一个bean 声明为一个切面-->
        <aop:aspect ref="logService">
            <!--定义一个切入点，切入点的ID，切入点要切的方法，使用了aspectJ表达式-->
            <aop:pointcut id="doSomething" expression="execution(* *.triggleDoSthImp(..))"/>

            <!--声明前置通知，对应了前面声明的切入点，达到切入点之前，运行该前置通知对应的方法-->
            <aop:before method="logBeforeOperatoin" pointcut-ref="doSomething"/>

            <!--声明后置通知，达到切入点之后，运行该后置通知对应的方法-->
            <aop:after method="logAfterOperatoin" pointcut-ref="doSomething"/>

        </aop:aspect>


    </aop:config>

</beans>

```
主类没有啥改变，最终实现了业务逻辑类毫无痕迹地对公共服务类的使用，代码解耦，业务核心代码也干净整洁

