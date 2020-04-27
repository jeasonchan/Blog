# 1 前言
Java并发编程：深入剖析ThreadLocal     https://www.cnblogs.com/dolphin0520/p/3920407.html

要点：
1、每一个线程对象（Thread）实例，都有一个非静态属性，threadLocals（一开始是null的），是一个Map<ThreadLocal,Object>类型的Map
2、ThreadLocal其实是一个数据容器，装在这个容器中的对象在get时都会从threadLocals中取初始值的拷贝使用
3、对ThreadLocal的实例调用set方法，设置希望被使用的初始值，之后在各个线程中get出来的就是最近set进去的对象的拷贝
