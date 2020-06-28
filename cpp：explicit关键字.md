# 1 前言

参考文档：

C++中explicit的用法     https://blog.csdn.net/qq_35524916/article/details/58178072

# 2 

C++提供了关键字explicit，可以阻止   经过转换构造函数进行的隐式转换   的发生，声明为explicit的**构造函数**不能在隐式转换中使用。

C++中，只有一个入参数的**构造函数**(或者除了第一个参数外其余参数都有默认值的多参构造函数也算)， 承担了两个角色：

1. 构造函数
2. 默认且隐含的类型转换操作符

所以，有时候在我们写下如 AAA = XXX， 这样的代码， 且恰好XXX的类型正好是AAA单参数构造器的参数类型时， 编译器就会自动调用这个构造器， 创建一个AAA的对象。

虽然这样看起来好象很方便，但在某些情况下， 却违背了程序员的本意。 这时候就要在这个构造器前面加上explicit修饰， 指定这个构造器只能被明确的调用/使用， **不能作为类型转换操作符被隐含的使用。** 

解析：explicit构造函数是用来防止隐式转换的。请看下面的代码：

```cpp
#include <iostream>
using namespace std;
class Test1
{

public :
  Test1(int num):n(num){}
  
private:
  int n;
};

class Test2
{
public :
  explicit Test2(int num):n(num){}
  
private:
  int n;
};

int main()
{
  Test1 t1 = 12;
  Test2 t2(13);
  Test2 t3 = 14;//这一行必然报错，因为Test2这个类并没有定义赋值函数

  return 0;
}

```

编译时，会指出 t3那一行error:无法从“int”转换为“Test2”。而t1却编译通过。注释掉t3那行，调试时，t1已被赋值成功。

注意：当类的声明和定义分别在两个文件中时，explicit只能写在在声明中，不能写在定义中。
