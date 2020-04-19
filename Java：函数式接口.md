# 1 前言

参考  菜鸟教程  https://www.runoob.com/java/java8-functional-interfaces.html



# 2 Java 8 函数式接口

## 2.1 定义函数式接口并实现

函数式接口(Functional Interface)就是一个**有且仅有一个抽象方法**，但是可以有多个非抽象方法的接口。

函数式接口可以被隐式转换为 lambda 表达式。

Lambda 表达式和方法引用（实际上也可认为是Lambda表达式）常用来隐式实现函数式接口。

可以按照如下方式，定义一个没有返回值的函数式接口：

```java
@FunctionalInterface
interface GreetingService 
{
    void sayMessage(String message);
}
```

其中，接口的注解@FunctionalInterface，JDK的官方注释是：

```
However, the compiler will treat any interface meeting the
 * definition of a functional interface as a functional interface
 * regardless of whether or not a {@code FunctionalInterface}
 * annotation is present on the interface declaration.
```

也就是，只要这个接口满足函数式接口的条件，编译器会自动识别为函数式接口，**有了该接口，只是更方便IDE帮我们检查当前接口是否符合函数式接口的条件。**

再来一定一个有返回参数的函数式接口：

```java
package default_package.函数式接口;

@FunctionalInterface
public interface ReturnGreeting {
    String greeting(String name);
}

```

现在，分别以匿名函数、lambda推断、lambda不推断的方式实现以上两个函数式接口：

```java
package default_package.函数式接口;

import java.sql.SQLOutput;

public class Main {
    public static void main(String[] args) {
        //适用匿名函数实现接口
        GreetService greetService = new GreetService() {

            @Override
            public void greet(String name) {
                System.out.println("匿名函数实现接口：");
                System.out.println("Hello " + name);
            }
        };

        greetService.greet("jeason");


        //使用lambda表达式实现接口
        //lambda能省略这么多是因为，输入输出能直接推断出来
        GreetService greetService1 = (name) -> System.out.println("Hello" + name);

        GreetService greetService2 = (name) -> {
            System.out.println("Hello" + name);
        };


        //因为接口中未定义的函数是唯一的，完全符合lambda表达式的唯一结果推断
        //单条语句时，甚至可以省略用于返回的return
        ReturnGreeting returnGreeting = (name) -> "Hello" + name;

        ReturnGreeting returnGreeting1 = (name) -> {
            return "Hello" + name;
        };


    }
}
```

## 2.2 JDK中的函数式接口

JDK 1.8 新增加了专门的函数包，包中的各个接口只有一个未实现的方法，其余的都式由default定义的方法。

常用的函数式接口有：

```java
Consumer<T>;
//代表了接受一个输入参数并且无返回的操作

Supplier<T>;
//无参数，返回一个结果。

Predicate<T>
//接口一个入参，返回布尔值
```



## 2.3 小结

无论是lambda还匿名函数，本质上都是生成一个对象，之后再调用生成对象实现的接口方法，实现相应的功能。

Lambda表达式具有的很强的推断能力，有返回值的函数式接口，某些情况下可省略return关键字。