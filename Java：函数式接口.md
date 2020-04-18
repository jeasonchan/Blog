# 1 前言

参考  菜鸟教程  https://www.runoob.com/java/java8-functional-interfaces.html



# 2 Java 8 函数式接口

函数式接口(Functional Interface)就是一个**有且仅有一个抽象方法**，但是可以有多个非抽象方法的接口。

函数式接口可以被隐式转换为 lambda 表达式。

Lambda 表达式和方法引用（实际上也可认为是Lambda表达式）常用来隐式实现函数式接口。

可以按照如下方式，定义一个函数式接口：

```java
@FunctionalInterface
interface GreetingService 
{
    void sayMessage(String message);
}
```

