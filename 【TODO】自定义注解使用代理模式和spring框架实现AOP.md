# 1 背景
自定义注解，并通反射进行手动解析：
[java注解:如何实现和使用一个自定义注解？](https://blog.csdn.net/wangpengzhi19891223/article/details/78131137)
[java自定义注解实现记录日志功能](https://blog.csdn.net/xl_1803/article/details/100583043)

（java原生实现）使用静态代理和动态代理：
[静态代理模式和动态代理模式实现AOP](https://www.cnblogs.com/pwc1996/p/4839150.html)

（Spring实现）直接注册AOP处理：

[Spring AOP 基于注解实现日志记录+自定义注解](https://blog.csdn.net/weixin_42184707/article/details/80348103)

# 2 自定义注解并手动解析

先回顾一下的注解的基本信，一共有四个元注解：

* @Document 

* @Document ，有的话就表明要包含到文档中

*  @Target ，表明当前注解适用的类型，比如，类、方法、构造函数、属性等**程序元素**

*  @Retention ，保留等级用：

  @Retention(RetentionPolicy.CLASS)修饰的注解，表示注解的信息被保留在class文件(字节码文件)中当程序编译时，但不会被虚拟机读取在运行的时候；
  用@Retention(RetentionPolicy.SOURCE )修饰的注解,表示注解的信息会被编译器抛弃，不会留在class文件中，注解的信息只会留在源文件中；
  用@Retention(RetentionPolicy.RUNTIME )修饰的注解，表示注解的信息被保留在class文件(字节码文件)中当程序编译时，会被虚拟机保留在运行时。 
  
* @Inherited，表明当前注解是否适用于类的继承

java注解的功能实现基本是通过定义属性实现的，然后，通过反射的方式，使用相应的处理类，处置注解中的属性，最终实现相应的功能。注解处理器类库是java.lang.reflect.AnnotatedElement，AnnotatedElement 接口是所有**程序元素**（Class<T>类、Method类和Constructor类）的父接口，所以程序通过反射获取了某个类的AnnotatedElement对象之后，程序就可以调用该对象的如下四个方法来访问Annotation信息：
* 方法1：<T extends Annotation> T getAnnotation(Class<T> annotationClass): 返回改程序元素上存在的、**指定类型**的注解，如果该类型注解不存在，则返回null。
* 方法2：Annotation[] getAnnotations():返回该程序元素上存在的所有注解。
* 方法3：boolean is AnnotationPresent(Class<?extends Annotation> annotationClass):判断该程序元素上是否包含**指定类型**的注解，存在则返回true，否则返回false.
* 方法4：Annotation[] getDeclaredAnnotations()：返回**直接存在**于此元素上的所有注释，即一眼就能看到的、写明的注解。与此接口中的其他方法不同，该方法将忽**略继承的注释**。（如果没有注释直接存在于此元素上，则返回长度为零的一个数组。）该方法的调用者可以随意修改返回的数组；这不会对其他调用者返回的数组产生任何影响。

## 2.1 自定义一个注解

```java
package default_package.自定义注解并解析;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)//适用于方法
@Retention(RetentionPolicy.RUNTIME)//在运行期时要进行使用
public @interface Record {
    String user();

    String action();

    String detail();

}

```

定义了注解，用于模拟AOP。

## 2.2 自定义的处理类

预期的使用方法如下：

```java
package default_package.自定义注解并解析;

public class Main {

    @Record(user = "admin", action = "delete", detail = "delete log")
    public void admin_delete_log() {
        System.out.println("admin is trying to delete log.");
    }


    public static void main(String[] args) {

    }
}

```

使用的方法是，被@Record注解的函数，被调用时，要写打印处注解里的所有的属性。以这个为出发点，写一下处理类，为了以后方便处理的这个Record注解，定义一个接口：

```java
package default_package.define_annotation_and_use;

public interface HandlerRecordAnnotation {
    void handlerRecordAnnotation();
}

```

然后，Main类实现这个接口，接口实现的具体内容就是，通过反射获取当前类中声明的方法，并在声明方法中找到有Record注解的方法，并打印该方法上Record的相关属性。最终，实现和运行结果如下:

```java
package default_package.define_annotation_and_use;

import java.lang.reflect.Method;


public class Main implements HandlerRecordAnnotation {

    @Record(user = "admin", action = "delete", detail = "delete log")
    public void admin_delete_log() {

        handlerRecordAnnotation();//没有面向切面，还是要手动出发一下

        System.out.println("admin is trying to delete log.");
    }


    @Override
    public void handlerRecordAnnotation() {
        Method[] methods = this.getClass().getDeclaredMethods();
        for (Method method : methods) {
            if (method.isAnnotationPresent(Record.class)) {
                Record record = method.getAnnotation(Record.class);
                System.out.println("Record annotation info:" +
                        record.user() + " " +
                        record.action() + " " +
                        record.detail());
            }
        }

    }


    public static void main(String[] args) {
        new Main().admin_delete_log();

    }
}

/*
运行结果：
Record annotation info:admin delete delete log
admin is trying to delete log.

Process finished with exit code 0

*/
```

# 3 原生实现AOP





# 4 Spring实现AOP