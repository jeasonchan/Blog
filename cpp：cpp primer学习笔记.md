# 1 开始

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout << "标准输出" << endl;
    cerr << "标准错误" << endl;
    clog << "运行时消息" << endl;

    cout << "输入两个数字：" << endl;

    int number1;
    int number2; //

    cout << "初始化为：" << number1 << " " << number2 << endl;

    cin >> number1 >> number2; //一直会阻塞直到输入了两个非空的字符

    cout << "number1:" << number1 << "  number2:" << number2 << " and sum is " << number1 + number2 << endl;

    cout << "/*" << endl;
    cout << "/*" << endl;

    // while语句
    int sum = 0, val = 1;
    while (val <= 10)
    {
        sum += val;
        val++;
    }
    cout << "sum=" << sum << endl;

    bool haha = true;

    sum = 0;
    val = 10;

    while (val >= 1)
    {
        sum += val;
        --val;
    }

    sum = 0;
    val = 50;
    for (; val > 0; ++val)
    {
        sum += val;
    }

    sum = 0;
    val = 0;
    //while的判断条件是 cin>>val  这个表达式的值
    //也就是  >>  这个函数的值，也就是标准输入流  cin
    //当cin遇到 文件结束符（eof） 或者 无效输入时，cin的状态为无效状态，即为false
    //其与可以人为时有效的
    while (cin >> val)
    {
        cout << "Got:" << val << endl;
        sum += val;
    }

    //想推出以上循环，就只能故意输进去字符，强制cin无效；或者，ctrl+z然后enter，输入文件结束符

    cout << "end and sum is " << sum << endl;

    return -1;
}
```

未初始化的变量：

基本数据类型声明时，确实会有一个初始值，但是，是不确定的初始值。类 类型的变量如果未指定初始值，就会自动调用无参构造（可能时编译器自动添加的无参构造）。类类型的成员变量，在为对象申请内存时就会按照类成员定义的先进行一波初始化，**然后才会开始**调用构造函数，所以，为了避免在构造函数中**二次初始化**自定义类型的成员变量，引入了参数列表。

```cpp
private:
    string s_;

public:
Base(const string& s) 
{ 
    s_.string::string(s); 
} 

/* 
真是的执行顺序是：

string s_;//在构造函数之前先执行

Base(const string& s) { 
    s_=s;//这是一个左值赋值函数，会不会有较大的性能开销看string的=实现
} 
*/

```

# 2 变量和基本类型
## 2.1 内置的基本类型
cpp和java一样，虽然都是面向对象的，本应该是一切皆对象（cpp其实可以多种，过程、对象、函数式，都可以），但还是有的旧时的影子，那就是基本数据类型。

cpp的基本数据类型，分为算术类型和空类型（void）。

### 2.1.1 算数型
算数型还分为：
* 整型，包括 bool、char、wchar_t、char16_t char32_t、short、int、long long
* 浮点型，包括 float、double、long double

每种类型的占用的比特位数稍有不同。其中最坑的莫过于字符相关的整型：char、wchar_t、char16_t、char32_t,稍有不慎就会发生截断，造成字符显示异常。

C++读写汉字，C++处理中文字符             https://blog.csdn.net/calmreason/article/details/7935258

C++11新特性--Unicode 16位和32位支持           https://www.jianshu.com/p/8b87a05c23b1

内置类型的机器实现：
P31