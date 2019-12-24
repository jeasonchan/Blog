@[TOC]
# 1 slf4j和log4j系列区别及联系
## 1.1 log4j系列
有log4j和log4j2，本身就是完成的一套日志记录、打印系统，完全可以单独使用log4j或者log4j2，比如：
单独使用log4j：
```java
public class Log4jTest {
	private static final org.apache.log4j.Logger logger = org.apache.log4j.Logger.getLogger(Log4jTest.class);
	public static void main(String[] args) {
		logger.info("hello word");
	}
}
```
单独使用log4j2：
```java
public class Log4j2Test {
	private static org.apache.logging.log4j.Logger logger = org.apache.logging.log4j.LogManager.getLogger(Log4jTest.class);
	public static void main(String[] args) {
		logger.info("hello word");
	}
}
```
两个日志的引用包路径不同，并且！配置方式不一样，log4j2对properties的配置支持不是很好，它的格式一般为xml格式或者yaml格式，这种格式的可读性比较好，各种配置一目了然
## 1.2 slf4j
slf4j仅仅是一个为Java程序提供日志输出的统一接口，并不是一个具体的日志实现方案，就比如JDBC一样，只是一种规则而已，所以单独的slf4j是不能工作的，必须搭配其他具体的日志实现方案，比如log4j或者log4j2。以log4j为例，需要引入下面的jar包：
* log4j核心jar包：log4j-1.2.17.jar
* slf4j核心jar包：slf4j-api-1.6.4.jar
* slf4j与log4j的桥接包：slf4j-log4j12-1.6.1.jar，这个包的作用就是使用slf4j的api，但是底层实现是基于log4j
使用代码如下：
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 public class Slf4jTest {
	private static final Logger logger = LoggerFactory.getLogger(Slf4jTest2.class);
 	public static void main(String[] args) {
		logger.info("hello world");
	}
}
```
或者直接使用lombok的@slf4j注解，然后直接使用"log.info("23333333333333333");"进行日志记录。
# 2 log4j日志等级
log4j定义了8个级别的log（除去OFF和ALL，可以说分为6个级别），优先级从高到低依次为：OFF、FATAL、ERROR、WARN、INFO、DEBUG、TRACE、 ALL。
* ALL 最低等级的，用于打开所有日志记录，很低的日志级别，一般不会使用。
* DEBUG 指出细粒度信息事件对调试应用程序是非常有帮助的，主要用于开发过程中打印一些运行信息。
* INFO 消息在粗粒度级别上突出强调应用程序的运行过程。打印一些你感兴趣的或者重要的信息，这个可以用于生产环境中输出程序运行的一些重要信息，但是不能滥用，避免打印过多的日志。
* WARN 表明会出现潜在错误的情形，有些信息不是错误信息，但是也要给程序员的一些提示。
* ERROR 指出虽然发生错误事件，但仍然不影响系统的继续运行。打印错误和异常信息，如果不想输出太多的日志，可以使用这个级别。
* FATAL 指出每个严重的错误事件将会导致应用程序的退出。这个级别比较高了。重大错误，这种级别你可以直接停止程序了。
* OFF 最高等级的，用于关闭所有日志记录。
如果将log level设置在某一个级别上，那么比此级别优先级高的log都能打印出来。例如，如果设置优先级为WARN，那么OFF、FATAL、ERROR、WARN 4个级别的log能正常输出，而INFO、DEBUG、TRACE、 ALL级别的log则会被忽略。Log4j建议只使用四个级别，优先级从高到低分别是ERROR、WARN、INFO、DEBUG，并且，**log4j默认的优先级为ERROR**。
# 3 打印等级控制
以dropwizard框架为例，在yml配置文件中配置日志的打印等级为DEBUG级别：
```yml
logging:
  level: DEBUG
  appenders:
    - type: TransactionConsole
      threshold: INFO
      logFormat: "%d{yyyy-MM-dd HH:mm:ss SSS} %-5p [%c][%t] - %m%n"
    - type: Sizefile
      threshold: INFO
      logFormat: "%d{yyyy-MM-dd HH:mm:ss SSS} %-5p [%c][%t] - %m%n"
      currentLogFilename: /home/log/app.log
      archivedLogFilenamePattern: /home/zip/app-%i.log.gz
      maxFileSize: 200MB
      archivedFileCount: 20
      timeZone: UTC
    - type: socketJson
      threshold: INFO
      autoDiscover: true
      logstashServiceName: commsrvlogstash
      logstashServiceVersion: v1
```

