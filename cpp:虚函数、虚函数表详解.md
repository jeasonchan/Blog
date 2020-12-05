# 1 背景
面试问到了，止只回答对了浅层次的东西，在此学习记录一下。

参考文档:

https://blog.csdn.net/primeprime/article/details/80776625

https://blog.csdn.net/wdkirchhoff/article/details/43823881

https://blog.csdn.net/qican_7/article/details/100603461

先简单书说一下重点结论:

1、虚函数表在编译期已经生成，和函数放在同一片内存区域，整个类共用一个虚函数表

2、编译器会为含有虚函数的类添加一个指向虚函数表的指针，这个指针在类实例初始化时自动指向类公用的虚函数表

3、子类重写父类的虚函数后，虚函数表中，函数的地址会指向子类自己实现的方法的地址，子类不重写则不会覆盖
