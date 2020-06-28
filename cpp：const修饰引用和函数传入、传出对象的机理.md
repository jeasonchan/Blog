# 1 前言

参考文章：

const介绍    https://light-city.club/sc/basic_content/const/

C++const类型的引用参数         https://www.cnblogs.com/a-lai/p/7338316.html

C++中的临时变量       https://blog.csdn.net/rongwenxiao/article/details/52193233

看const关键字；其中介绍到const修饰函数入参时，有一个重点是，const类型的引用参数；再根据const类型的入参延伸到cpp中的"临时变量"概念。

cpp为了保证性能，调用函数时应该尽量避免直接传递"对象"，因为传递对象会有存在对象拷贝过程；因此，在内存安全的前提下，应该尽量使用 引用/指针 作为函数的形参，以提高程序的性能。

# 2 const类型的引用参数







# 3 对象作为函数的入参和返回值 的机理探索

在函数定义时，如果函数入参和返回值，定义的就是对象，而不是指针或者引用，比如：

```cpp
Person getInstance();

void print(Person person);
```

则，从函数定语义中就可以看出，函数定义的人就是希望入参和返回值是一个对像，尽管可以这样定义，但是并不合理！因为编译器会为这种定义、调用的点偷偷进行优化，优化为"引用+临时变量+拷贝"的方式。

临时变量就是暂时没有姓名的变量，就是右值，和左值相对应。典型的右值就比如："123",123 等等。
