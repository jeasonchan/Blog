# 1 前言
参考文章：

maven之打包插件（maven-assembly-plugin，maven-shade-plugin与maven-assembly-plugin）    https://yq.aliyun.com/articles/308777


JAR包中的MANIFEST.MF文件详解以及编写规范   https://www.cnblogs.com/EasonJim/p/6485677.html



运行java程序本质上就是一句命令行，即：

```bash
# 下文称为java命令行
java [options] [full class name] [......args]
````

要使程序运行起来，必须满足两个条件：

1. 存在程序入口：public static void main(String[] args)
    
对于任意一个jar包或者一堆jar包，要想程序基于这些jar包（其实是基于其中的字节码）运行起来，需要一个启动入口，即为主函数，主函数所在的类即为主类，因此，使用java命令行启动程序时，必须要指明主类。

指明主类的方式有两种：

* 命令行中指明主类，如：

```bash
java -jar Hello.jar com.jeasonchan.Main
```

* 在jar包中的mainfest中声明Main-Class
    
详见参考文章的链接。


当mainfest和命令行执行的主类不同时，以命令行指定的为准。


2. 能找到需要的类

类是否能找到，就是看能否通过package（本质就是一层层文件夹）和类名找到相应的字节码文件。

package中的一层层文件夹其实就是相对路径，这个相对路径的根路径就是众多jar包进去之后的地方，这些jar包必定是来自classpath。

可以用"java -h"查看一下classpath的意义,如下：
```
-classpath <目录和 zip/jar 文件的类搜索路径>
```
这个参数可以是文件夹、zip包、jar包。

使用命令行运行时，会自动添加JDK的字节码文件/文件夹到classpath，因此，只需要将自己依赖的jar包通过某种方式加入到classpath中，和指定主类一样，有两种方式：

* 通过命令行参数 -classpath添加

添加多个文件、jar包、zip包时，使用英文分号进行分割

* 在jar包的mainfest中申明额外的需要的jar包，如：
```
<!-- MAINFEST.MF -->
Class-Path: commons-beanutils.jar commons-collections.jar commons-digester.jar commons-logging.jar commons-validator.jar jakarta-oro.jar struts-legacy.jar
```
在MAINFEST中声明classpath时，说明这些被声明的jar包并没有以字节码的形式和当前的jar包中的字节码合为一体，而是以独立jar包的形式和当前jar包并存。这种方式有个缺点，文件太多了，不好管理。


可见，声明入口类和classpath都有命令行和MAINFEST两种方式，而MAVEN中提供的打包、运行方式无非就是对这两种方式的组合运用。

下面举例说明常见的组合运用方式和自己可以用的方式。


# 2 运用实践

## 2.1 IDEA的Run按钮
在idea中写一个简单的应用，点击Run按钮，可以看到终端会打印出Run按钮实际执行的命令行：

```bash
C:\CRsoftwares\Java\jdk-11.0.1\bin\java.exe 

-javaagent:C:\CRsoftwares\Toolbox\apps\IDEA-U\ch-0\193.5233.102\lib\idea_rt.jar=53681:C:\CRsoftwares\Toolbox\apps\IDEA-U\ch-0\193.5233.102\bin 

-Dfile.encoding=UTF-8 

-classpath C:\CRroot\documents\codeproject\java_exercise_based_on_idea\java_exercise\target\classes;C:\Users\chenr\.m2\repository\org\apache\commons\commons-compress\1.8\commons-compress-1.8.jar;C:\Users\chenr\.m2\repository\org\tukaani\xz\1.5\xz-1.5.jar;C:\Users\chenr\.m2\repository\org\projectlombok\lombok\1.18.8\lombok-1.18.8.jar;C:\Users\chenr\.m2\repository\org\slf4j\slf4j-simple\1.7.26\slf4j-simple-1.7.26.jar;C:\Users\chenr\.m2\repository\org\slf4j\slf4j-api\1.7.26\slf4j-api-1.7.26.jar;C:\Users\chenr\.m2\repository\com\h2database\h2\1.4.199\h2-1.4.199.jar;C:\Users\chenr\.m2\repository\com\fasterxml\jackson\core\jackson-core\2.8.0\jackson-core-2.8.0.jar;C:\Users\chenr\.m2\repository\com\fasterxml\jackson\core\jackson-databind\2.8.0\jackson-databind-2.8.0.jar;C:\Users\chenr\.m2\repository\com\fasterxml\jackson\core\jackson-annotations\2.8.0\jackson-annotations-2.8.0.jar;C:\Users\chenr\.m2\repository\com\google\guava\guava\18.0\guava-18.0.jar;C:\Users\chenr\.m2\repository\cglib\cglib\rc2-1.0\cglib-rc2-1.0.jar;C:\Users\chenr\.m2\repository\mysql\mysql-connector-java\8.0.19\mysql-connector-java-8.0.19.jar;C:\Users\chenr\.m2\repository\com\google\protobuf\protobuf-java\3.6.1\protobuf-java-3.6.1.jar;C:\Users\chenr\.m2\repository\org\apache\zookeeper\zookeeper\3.6.1\zookeeper-3.6.1.jar;C:\Users\chenr\.m2\repository\commons-lang\commons-lang\2.6\commons-lang-2.6.jar;C:\Users\chenr\.m2\repository\org\apache\zookeeper\zookeeper-jute\3.6.1\zookeeper-jute-3.6.1.jar;C:\Users\chenr\.m2\repository\org\apache\yetus\audience-annotations\0.5.0\audience-annotations-0.5.0.jar;C:\Users\chenr\.m2\repository\io\netty\netty-handler\4.1.48.Final\netty-handler-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\io\netty\netty-common\4.1.48.Final\netty-common-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\io\netty\netty-resolver\4.1.48.Final\netty-resolver-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\io\netty\netty-buffer\4.1.48.Final\netty-buffer-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\io\netty\netty-transport\4.1.48.Final\netty-transport-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\io\netty\netty-codec\4.1.48.Final\netty-codec-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\io\netty\netty-transport-native-epoll\4.1.48.Final\netty-transport-native-epoll-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\io\netty\netty-transport-native-unix-common\4.1.48.Final\netty-transport-native-unix-common-4.1.48.Final.jar;C:\Users\chenr\.m2\repository\org\slf4j\slf4j-log4j12\1.7.25\slf4j-log4j12-1.7.25.jar;C:\Users\chenr\.m2\repository\log4j\log4j\1.2.17\log4j-1.2.17.jar;C:\Users\chenr\.m2\repository\org\apache\curator\curator-recipes\4.3.0\curator-recipes-4.3.0.jar;C:\Users\chenr\.m2\repository\org\apache\curator\curator-framework\4.3.0\curator-framework-4.3.0.jar;C:\Users\chenr\.m2\repository\org\apache\curator\curator-client\4.3.0\curator-client-4.3.0.jar 

default_package.实验指定Main.Main
```

将以上命令进行精简分析，剔除一些和本文无关的参数，也就是：

```bash
java

-classpath 

C:\CRroot\documents\codeproject\java_exercise_based_on_idea\java_exercise\target\classes;C:\Users\chenr\.m2\repository\org\projectlombok\lombok\1.18.8\lombok-1.18.8.jar;

default_package.实验指定Main.Main
```
也就是命令行中添加了编译后的字节码文件夹"xxxxxxxx/target/classes"和pom中声明的依赖在本地的jar包。

可见，idea的Run按钮，就是完全以java命令行的方式添加依赖、声明主类。

在maven没有出来之前，基本都是以这种方式运行程序，依赖的jar包较多时，就将jar放到自己建立的lib文件夹，classpath就直接添加该lib文件夹即可。

## 2.2 推测Maven的编译、执行
maven出现后，极大的方便管理依赖、编译和执行。以下开始对maven的编译、打包、执行进行推测，并不代表maven的真正的机理！！！！

编译、打包、运行。

编译的话，就是把java文件编译为字节码文件。

在不使用额外插件的情况下，也不作额外配置的情况下，对当前工程编译、打包时，只会把自己写的java源码打包进一个jar包（假设叫Hello.jar），mainfest会添加依赖的jar名到classpath，但mainfest不会有主类名。如果直接用：

```bash
java -jar Hello.jar
# 因为mainfest中没有声明主类，一开始可能就会报找不到主类

java -classpath Hello.jar com.jeasonchan.Main
# 主类能找到了，因为通过命令行进行声明了；但是，必然会被xxxxxx类没有找到
# 因为，mainfest声明了依赖的jar包的，
# 但是，classpath里并不能找到依赖的jar包里的字节码文件

java -classpath Hello.jar;/javalibs; com.jeasonchan.Main
# 将依赖的jar包都放进  /javalibs，应该就能运行了
```

如果使用了maven的shade打包插件，编译、打包过程应该会发生变化。在仅仅引入了shade插件，但不做任何配置的情况下（就是，不在pom中指明mianClass）。编译打包的过程应该是：

1. 将java源码编译为字节码，各个字节码文件产生在各自声明的package路径中
2. maven将依赖的jar包中的文件都提取出来，和上面的字节码进行合并
3. 将合并后的有层次的字节码文件合集，打包为一个jar包，mainfest不会声明classpath（因为依赖的jar包里的的字节码文件已经和自己字节码合并在一起了），也不会声明主函数（因为，自己根本没配置）

如果通过以命令行运行：

```bash
java -classpath Hello.jar com.jeasonchan.Main
# 应该能直接运行，入口类和依赖的jar包的字节码都已经有了；

java -jar Hello.jar
# 应该会报找不到主类


# 如果，在shade的配置里，声明主类时 com.jeasonchan.Main
# 主类信息会写进 mainfest 中
java -jar Hello.jar
# 完美运行，通过mainfest找到了主类，依赖的字节码文件也都有
```


## 2.3 日常自己可用的方式

1. maven使用shade插件，不必做任何配置，利用其字节码合并打包的特性
2. 命令行运行时，手动指定主类即可