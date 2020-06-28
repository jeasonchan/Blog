@[TOC]
# 1后置处理器是什么？
一个类，实现了接口BeanPostProcessor 定义的两个方法，这两个方法分别是：postProcessBeforeInitialization和postProcessAfterInitialization，顾名思义，就是分别在bean的init-method前后进行分别执行这两个方法。
**多后置处理器的有序性的**
bean在使用的过程中可以经过多个后置预处理的处理，但是，一般情况下，多个实现后置处理器接口的类是有先后顺序的，为了让IOC明白后置处理器之间的先后顺序，类还要实现Ordered接口，通过接口里的order属性来控制后处理器的先后顺序，默认为0，为最高优先级。
**同一个容器中的后置处理器是通用的**
一个context中的后置处理器实现类不是针对某一个的bean，这个context中的所有bean的产生过程都回去调用这个后置处理器，为了有针对性，可以通过bean的id来执行特异话的操作。
# 2 后置处理器实现类
BeanPostProcessor 接口源码
```java
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
```

可见，返回值必须为非null，且必须是Object类型，和从context中get出来的一致。由于是有默认实现，因此，不是必须要重写方法的。现在看一个后处理器类的实例：
```java
package 后置处理器;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class PostProcessorExample implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if ("chinese".equals(beanName)) {
            System.out.println("后置处理器，初始化前处理的bean的id是：" + beanName);
        } else {
            System.out.println("后置处理器，初始化前处理的bean的id是：" + beanName);
        }
        return bean;
    }

}
```

通过beanName对处理的bean进行针对性的区分、处理。
# 3 后置处理器实现类bean配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="chinese" class="初始化和销毁回调.Chinese">
    </bean>

    <bean id="american" class="初始化和销毁回调.American">
    </bean>

    <bean id="beanPostProcessor" class="后置处理器.PostProcessorExample">
    </bean>

</beans>
```
# 4 运行调用
主类
```java
package 后置处理器;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;

public class PostProcessorExample implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if ("chinese".equals(beanName)) {
            System.out.println("后置处理器，初始化前处理的bean的id是：" + beanName);
        } else {
            System.out.println("后置处理器，初始化前处理的bean的id是：" + beanName);
        }

        return bean;
    }

}
```
其中America类实现了InitializingBean, DisposableBean这两个接口，xml配置元数据中不配置也自带初始化和销毁方法，Chinese完全没有指定初始化和销毁方法，因此，控制台输出是：
```bash
后置处理器，初始化前处理的bean的id是：chinese
后置处理器，初始化前处理的bean的id是：american
American 初始化回调函数
American 销毁回调函数
```
# 5多后置处理器调用
最理想的情况下，让实现类去实现Ordered最理想的，但是，在默认情况下，Spring容器会根据后置处理器的定义顺序来依次调用，比如：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://www.springframework.org/schema/beans    http://www.springframework.org/schema/beans/spring-beans.xsd">
  <!-- bean定义 -->  
  <bean id="narCodeService" class="com.test.service.impl.NarCodeServiceImpl">
  </bean>
  <bean id="postProcessor" class="com.test.spring.PostProcessor"/>
  <bean id="postProcessorB" class="com.test.spring.PostProcessorB"/>
</beans>
```
在bean配置元数据，就回让IOC容器最后再调用B后置处理器。但是！官方还是建议去实现顺序接口来显式定义的后置处理器的实现顺序！
