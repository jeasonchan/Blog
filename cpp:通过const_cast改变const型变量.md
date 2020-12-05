# 1 背景
面试时被问到了能否在const函数里改变const类型变量的值，我说的从来没尝试过，然后又问我如果用const_cast能不能修改成功……我觉的应该不能成功，要不然就违背const的语义了，在此学习记录一下const_cast。

首先const函数内部肯定是只能读所有的成员变量，不能有对成员变量的赋值操作，且也只能调用const函数，因为const函数有改变成员变量的潜在风险。

那实在想改const成员变量有没有什么办法呢?

参考文章:

https://www.cnblogs.com/zhuwbox/p/3383188.html

https://www.cnblogs.com/lvdongjie/p/5249729.html

结论:

1、一般情况下，直接通过const变量的变量去读变量的值时，都是直接走符号表的，不会再去从内存里读。所以，就算成功const_cast转换了对应的指针类型，也毫无卵用，即使指针地址相同的，但是，通过变量名读取时直接走符号表。

2、对于volitate const 这种修饰类型，是否修改成功，看编译期优化行为

3、不要尝试去改const变量
