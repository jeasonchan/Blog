# 1 背景

参考文档：

const char*, char const*, char*const 的区别    https://www.runoob.com/w3cnote/const-char.html

# 2 正文
cpp的爸爸Bjarne在他的The C++ Programming Language里面给出过一个助记的方法：

1. 把一个声明从右向左读
2. \* 理解为 pointer to


接下来举例：

```cpp
//类型1
//cp is a const pointer to char 
char * const cp;

//类型2
//p is a pointer to char const; 
const char * p; 

//类型3
//p is a pointer to const char
char const * p; 
```

类型1是顶层常量，指针本身不能变，永远只能只想一个地址。

类型2和类型3是完全等价的，因为，C++标准规定，const关键字放在类型或变量名之前等价的，且C++里面没有const*的运算符，所以const只能属于前面的类型。


