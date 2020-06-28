@[TOC](微服务框架)

# Dropwizard
## 杂七杂八
1. 目前遇到的参数有两种，一种是路径式参数，一种是赋值式参数，定义和使用方法如下：
```java
//待补充  赋值参数  https://blog.csdn.net/sunquan291/article/details/82530856
```
2. 【todo】 POJO放到javax.ws.rs.core.Response中时能自动序列化成json串吗，无需重写toString方法，也无需@JsonProperty注解，其余地方暂不确定是否需要注解
3. dropwizard-core模块和应用启动分析 - pinezhang - 博客园#360浏览器极速版# <https://www.cnblogs.com/ilovena/p/9864836.html>
4. yml文件中配置的地址是否已经是持久化卷？如何区分写的是持久化卷还是容器自己临时文件里面？
答：只从yml看无法区别，主要看蓝图里面的挂载点的配置，配置了的挂载点，yml里配置的路径才有可能挂载到持久化卷，否则永远只是在容器本身的系统里。容器一挂掉，文件会全部丢失。
5. 使用hibernate validator出现上面的错误， 需要注意，@NotNull 和 @NotEmpty  和@NotBlank 区别：
@NotEmpty 用在集合类上面
@NotBlank 用在String上面
@NotNull    用在基本类型上
6. dropwizard yml中的server、logging映射到什么类了？
答：由于我们自己写的configuration是继承自框架中的配置类，这个配置类中还有很多成员属性，从yml中获取的server、logging就是映射成configuration这个父类中的成员变量
## 概括
基于maven，将Web应用程序所需的所有依赖框架都结合在一起，大大缩短开发时间。其中用到的框架有：
* 用于HTTP的Jetty
Dropwizard项目没有将应用程序交给复杂的应用程序服务器，而是有一个main方法运行HTTP服务器，服务能像进程一样被任务管理器管理。HTTP服务器使用Jetty实现，Jetty 是一个开源的servlet容器，是基于Java的web容器。Jetty是使用Java语言编写的，它的API以一组JAR包的形式发布。开发人员可以将Jetty容器实例化成一个对象，为一些独立运行（stand-alone）的Java应用提供网络和web连接。
* 用于REST的Jersey
为了构建RESTful Web应用程序，需要JAX-RS的实现框架，dropwizard使用Jersey。Jersey可以让我们编写干净、可测试的类，这些类将HTTP请求映射成对POJO的操作。支持流输出，矩阵URI参数，条件GET请求等等。
* 用于JSON的Jackson
Jackson是将类实例和java基本数据容器比如list、map等，序列化和反序列化的工具。在本地使用JSON的序列化和反序列化<https://segmentfault.com/a/1190000005717319>、<https://www.cnblogs.com/winner-0715/p/6109225.html>，在dropwizard中注解，进行序列化和反序列化<https://blog.csdn.net/steven2xupt/article/details/80055169>
* 用于监控的Metrics
监控生产环境中的代码的行为。
* 其他集成的工具，Dropwizard还包括一些库：
Guava除了高度优化的不可变数据结构外，还提供了越来越多的类来加速Java的开发。
Logback和slf4j用于高性能和灵活的日志记录。
Hibernate Validator是JSR 349参考实现，它提供了一个简单的声明性框架，用于验证用户输入并生成有用且易于i18n的错误消息。
在Apache的HttpClient的和Jersey客户端库允许都与其他Web服务低收入和高层次的互动。
JDBI是使用Java的关系数据库最直接的方法。
Liquibase是在整个开发和发布周期中检查数据库模式的好方法，应用高级数据库重构而不是一次性DDL脚本。
Freemarker和Mustache是简单的模板系统，适用于面向用户的更多应用程序。
Joda Time是一个非常完整，理智的库，用于处理日期和时间

## yml配置文件解析
<http://ju.outofmemory.cn/entry/125442>

## 基本步骤
第一步，创建configuration类
第二步，以configuration类为参数创建application类
第三步，创建representation类（表示类或者叫暴露类）
第四步，创建资源类，由于rest风格编程都是针对资源展开的，每一个资源类和一个URI（统一资源标志）模板关联。Jersry将生成的资源（类实例还是什么的），寻找能将该资源转化为json的类，此此种类叫 provider类，能够产生/解读 java类生成的json，客户端收到provider生成的包含有application/json类型的资源
第五步，启动类，注册资源；将资源实例化，并注册到环境中，在此步实现了将默认配置注入到资源中。
第六步，健康检查；将健康检查实例注册到环境中

DropWizard结构的Web服务组成 
1. Configuration：用于设置该服务的配置，比方说在服务开放在哪个端口，数据库配置是怎样的等等。 
2. Application（Service）：该服务的主入口，定义该服务使用哪个配置文件，开放哪些Resource，该服务需要哪些HealthCheck等等。 
3. Resource：定义一个资源，包括如何获取该资源，对该资源做Get/Post/Delete/Query时，对应的各种业务逻辑。 
4. Representation：定义了一个服务返回值对象，当服务返回该对象时，会自动的把该对象按属性值生成一个Json格式的字符串返回给服务调用者。
5. HealthCheck：在DropWizard为每个服务提供的OM框架中用到，通过它可以随时检测当前服务是否可用。 
把该服务涉及的配置Configuration，资源Resource(资源和URI模板关联，URI映射Representation服务返回值对象)，HealthCheck统一整合到Application(Service)主类中：

## 常用注解
post
get
delete
query
path
consumes
produces
encoded
pathparam
contex


 ---
# Spring
 


---
# swagger


