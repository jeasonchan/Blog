#  1 背景





# 2 大致流程（以切面REST接口记日治为例）

先在pom中导入依赖：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```



首先定义一个接口，来表明，该接口需要记录日志：

```java
package com.jeasonchan.aopexercise.controller.Annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Log {

    //记录的接口的名字
    String restApiName();

    //记录一些别的信息，比如目标数据库啥的
    String detail();
}
```

在REST接口上使用：

```java
package com.jeasonchan.aopexercise.controller;

import com.jeasonchan.aopexercise.controller.Annotation.Log;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
public class ActionAndObject {

    @PostMapping("/log/{user}")
    @Log(restApiName = "logSth",detail = "haha")
    public Map<String, Object> logSth(@PathVariable("user") String user,
                                      @RequestParam("action") String action,
                                      @RequestBody(required = false) List<Object> objectList) {


        Map<String, Object> result = new HashMap<>();
        result.put("user", user);
        result.put(action, objectList);
        System.out.println(result);
        return result;


    }


}
```



为了实现注解的功能，要定义一个注解处理器，也就是要定义一个切面：

```java
package com.jeasonchan.aopexercise.controller.aop;


import com.jeasonchan.aopexercise.controller.Annotation.Log;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.Signature;
import org.aspectj.lang.annotation.*;
import org.aspectj.lang.reflect.MethodSignature;
import org.springframework.stereotype.Component;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import javax.servlet.http.HttpServletRequest;
import java.lang.reflect.Method;

//要使类称为切面处理类，必须满足两点：1、能注入到IOC中；2、被标注为aspect，成为切面类
@Component
@Aspect
public class LogAspect {

    //切入点是这个注解，但切入点实际上还是被注解注释的方法，因为，Spring的AOP的切入粒度都是方法这个粒度
    @Pointcut("@annotation(com.jeasonchan.aopexercise.controller.Annotation.Log)")
    public void pointCut() {
        //此处用这个方法代表Log注解的使用处
        System.out.println("line 18");
        //这一行打印并没有出现，并且IDEA提示，pointcut方法体应该为空
    }


    @Before(value = "pointCut()")
    public void doBefore(JoinPoint joinPoint) {
        System.out.println("line 26:" + joinPoint);
        //todo 为什么切点写明的是注解类型。joinpoint却是下面的被注解注释的方法？？？
        //line 26:execution(Map com.jeasonchan.aopexercise.controller.ActionAndObject.logSth(String,String,List))

        //todo 这个参数是哪里传进来的？？？？
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();

        System.out.println(request.getServletContext());

        handleJoinPoint(joinPoint);
    }


    //返回后通知，可以获取方法的返回值
    //使用前要先在注解中声明，才可以在返回后通知的方法中使用
    @AfterReturning(value = "pointCut()", returning = "controllerResult")
    public void doAfter(JoinPoint joinpoint, Object controllerResult) {
        System.out.println("line 33:" + joinpoint);
        //line 33:execution(Map com.jeasonchan.aopexercise.controller.ActionAndObject.logSth(String,String,List))

        handleJoinPoint(joinpoint);

        System.out.println("rest接口的返回值是：" + controllerResult);


    }


    private void handleJoinPoint(JoinPoint joinPoint) {
        Signature signature = joinPoint.getSignature();
        MethodSignature methodSignature = (MethodSignature) signature;
        Method method = methodSignature.getMethod();
        Log log = method.getAnnotation(Log.class);
        System.out.println(log.restApiName() + "  " + log.detail());


        System.out.println("=================================");


    }


}


```

最终，使用postman向接口发送消息时，控制台打印为：

```
line 26:execution(Map com.jeasonchan.aopexercise.controller.ActionAndObject.logSth(String,String,List))
org.apache.catalina.core.ApplicationContextFacade@220d7d8e
logSth  haha
=================================
{fuck=[XXX], user=jeasonchan}
line 33:execution(Map com.jeasonchan.aopexercise.controller.ActionAndObject.logSth(String,String,List))
logSth  haha
=================================
rest接口的返回值是：{fuck=[XXX], user=jeasonchan}
```

#  3 AOP注解详解

推荐阅读顺序：

* [Spring AOP详解](https://www.cnblogs.com/hongwz/p/5764917.html)  ，后面基于xml的AOP配置可直接跳过
*   [Spring boot中使用aop详解](https://www.cnblogs.com/bigben0123/p/7779357.html) 
* [@Pointcut()的execution、@annotation等参数说明](https://blog.csdn.net/java_green_hand0909/article/details/90238242)