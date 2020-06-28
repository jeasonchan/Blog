@[TOC]
# 功能
synchronized是Java中的关键字，是一种同步锁，保证某些“东西”同一时间只能有一个“人”能够“接触”到，它修饰的对象有下文几种。
1. 修饰一个代码块，被修饰的代码块称为同步语句块，同步句块的范围是大括号{}括起来的代码， 锁的范围根据synchronized("东西"){ xxxxxxxxxx }的括号中的“东西”来定，可以是this、类、变量/具体的实例，比如：
```java
class 人{
	public static String 筷子="筷子";

	//如果是这个，那么说明每个人的实例只能一顿一顿吃，不能同时（多线程）吃好几顿饭。
	synchronized(this){
	吃饭();
	}
	
	//如果是这个，那么只要整个人类有一个人在吃饭，其他所有人都不能吃饭，得等那个正在吃饭的人吃完。
	synchronized（人.class){
	吃饭（）；
	}

	//取得筷子权限才可以吃饭，被synchronized(筷子)修饰的代码块是互斥的
	synchronized(筷子){
	吃饭();
	}
}
```
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是这个方法依附的实例对象； 
```java
public synchronized void method()
{
   // todo
}
```
完全等价于下面的写法，和“补充”中的第一条相符！并且更推荐此种方法，效率更高。
```java
public void method()
{
   synchronized(this) {
      // todo
   }
}
```
3. 修饰一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 
```java
public synchronized static void method() {
   // todo
}
```
# 重要区别：synchronized和static synchronized
在都只是用来修饰某个方法的情况下，synchronized是对类的当前的这个实例进行加锁，防止其他线程同时访问这个实例的所有synchronized块，这个实例的所有synchronized块同时只有一部分是能被访问的。
static synchronized是限制线程同时访问jvm中该类的所有实例同时访问对应的代码块，同理，这个类的所有static synchronized块同时只有一部分是能被访问的。
综上：synchronized和static synchronized是两把毫不相干的锁，并且会对其他同类型的加锁区域互斥。
例子如下：
```java
pulbic class Something(){ 
	//某一个实例的方法锁
    public synchronized void isSyncA(){}
    public synchronized void isSyncB(){}
 	//类的静态方法锁
    public static synchronized void cSyncA(){}
    public static synchronized void cSyncB(){}
}
```
**注：该列子出自-结成浩的《java多线程设计模式》**
假设有Something类的两个实例x与y，那么分析下面的调用情况：
1. x.isSyncA()与x.isSyncB() 
不能同时执行这个语句，同一个实例的同步方法是互斥

2.  x.isSyncA()与y.isSyncA()
可以同时执行，不同实例的同步方法毫不关系，在内存里八竿子打不着

3. x.cSyncA()与y.cSyncB()
不能同时执行，同一个类的静态锁方法是互斥的

4. x.isSyncA()与Something.cSyncA()
可以同时执行，不通类型（static和非static）的锁，毫无关系

# 补充
1. synchronized methods( 参数 ){ xxxxxxx }与 synchronized(this){ xxxxxx }之间没有什么区别，只是 synchronized methods( 参数 ){ xxxxxx } 便于阅读理解，而 synchronized(this){ xxxxxx }可以更精确的控制冲突限制访问区域，有时候表现更高效率。也正是因为互斥锁是基于this来判断的互斥，因此，其他的synchronized(this)修饰的方法都是互斥的。
2. 关键字是不能继承的，也就是说，基类的方法synchronized f(){} 在继承类中并不自动是synchronized f(){}，而是变成了f(){}。继承类需要你显式的指定它的某个方法为synchronized方法；
3. 非常重要的！！synchronized的锁对象，十分强烈建议是final修饰的，而被final修饰的变量/引用，无法再使用赋值符号（“=”）对其二次赋值，即，指向的内存地址只能是一开始的地址。对于普通基础变量，用final修饰意味着不能再变；而对于对象引用，用于对象的还可以通过一些方法改变其中的成员变量值，因此，在一定程度上还是可变的，比如，如果将list作为锁定对象，任然既可以通过add、remove操作对lsit进行修改。

参考文章：Java中Synchronized的用法 <https://blog.csdn.net/luoweifu/article/details/46613015>
