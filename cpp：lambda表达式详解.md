# 1 背景

参考文档：

微软cpp文档（lambda的基本语法）  https://docs.microsoft.com/zh-cn/cpp/cpp/lambda-expressions-in-cpp?view=vs-2019

lambda表达式作为函数参数：https://blog.csdn.net/DumpDoctorWang/article/details/81903140


Lambda转换成函数指针  https://blog.csdn.net/u011680671/article/details/99660594

std::functional简单应用  https://www.jianshu.com/p/3d6a6578d7d4

头文件functional中的function泛型类解析   https://www.cnblogs.com/jerry-fuyi/p/11248665.html


微软文档（lambda与函数对象与函数指针）   https://docs.microsoft.com/zh-cn/cpp/cpp/lambda-expression-syntax?view=vs-2019    https://docs.microsoft.com/zh-cn/cpp/standard-library/function-objects-in-the-stl?view=vs-2019

微软文档（Lambda 表达式的示例）    https://docs.microsoft.com/zh-cn/cpp/cpp/examples-of-lambda-expressions?view=vs-2019


再开一个讨论区，C++11的std::function和template function的比较？ - 二律背反的回答 - 知乎
https://www.zhihu.com/question/41684177/answer/91952216


lambda闭包、捕获时的注意点     https://blog.csdn.net/guotianqing/article/details/103843256

# 2 lambda表达式定义语法
先来看一下cpp中lambda表达式的样子：

```cpp
#include <algorithm>
#include <cmath>

void abssort(float* x, unsigned n) {
    std::sort(x, x + n,
    
        // Lambda expression begins
        [](float a, float b) {
            return (std::abs(a) < std::abs(b));
        } // end of lambda expression

    );
}
```

std::sort函数，一共接受了三个参数，第三个参数是lambda表达式，和java中一样，lambda表达式本质是匿名对象，对象类型是匿名对象的类名，也就是随机字符串。但是！lambda仍然可以通过function泛型类进行精确定义、接受。

介绍一下cpp中lambda的各组成部分介绍：

```cpp
[=] (int x,int y) mutable throw() -> int 

{
    return x+y;
}
```

按照顺序对各部分进行介绍：

1. [=],捕获子句，CPP规范中叫 lambda引导，lambda函数体中仅能使用以下范围的对象：捕获子句中捕获的对象+lambda函数入参+lambda函数体中自己定义的对象+一些全局独对象

2. (int x,int y)，lambda函数的参数列表

3. mutable，可变规范，和普通函数中的一致

4. throw()，异常规范，和普通函数中的一致

5. -> int，返回值，普通函数中声明的位置不太一样

6. {return x+y;}，lambda表达式的函数体

接下来按lambda各部分分开介绍。

## 2.1 捕捉子句
```cpp
# 不进行任何捕获
[]

# 以 值 的方式捕获 只捕获 lambda 中提到的变量
[=]

# 以 引用 的方式捕获 只捕获 lambda 中提到的变量
[&]

# 捕获的方式（值、引用）、捕获的对象都不能重复
# 引用捕获和值捕获
[&total, factor]
# 值捕获和引用捕获
[factor, &total]

# 除了factor是值捕获，其余都是引用捕获
[&, factor]
[factor, &]

# 除了total是引用捕获，其余都是值捕获
[=, &total]
[&total, =]

[&, i]{};      // OK
[&, &i]{};     // 错误，已经声明了都是用引用捕获，没必要单独对i再声明一遍
[=, this]{};   // 错误，已经声明了都是用值捕获，没必要单独对this变量声明一遍

[=, *this]{ }; // OK，我也不知道为毛可以……

[i, i]{};      // 错误，没必要对i声明两次

# 在 C++14 中，可在 Capture 子句中引入并初始化新的变量，而无需使这些变量存在于 lambda 函数的封闭范围内。 初始化可以任何任意表达式表示；且将从该表达式生成的类型推导新变量的类型。 此功能的一个好处是，在 C++14 中，可从周边范围捕获只移动的变量（例如 std::unique_ptr）并在 lambda 中使用它们。
auto a = [ptr = move(pNums)]()
        {
           // use ptr
        };
```

注意点：
1. 以引用捕获外部对象时，lambda表达式内和外部，对对象的修改是相互影响的，要注意多线程问题
2. 以值捕获外部对象时，那必然是相互独立的（属性没有共享指针的情况下）

## 2.2 参数列表


## 2.3 可变规范


## 2.4 异常规范


## 2.5 返回类型



## 2.7 lambda体


# 3 lambda表达式作为函数入参
java中，lambda表达式看上去像个其实，其实是一个匿名对象，cpp的lambda表达式是不是也是这样呢？

一般人为了省事，定义和使用lambda表达式是都直接这样：

```cpp
 auto add_num1_and_num2 = [](int num1, int num2) -> int {
            return num1 + num2;
        };

cout << (add_num1_and_num2)(1, 2) << endl;
```

但是，函数入参不能使用auto类型作为入参类型，于是就项使用typeid函数看看lambda表达式的类型究竟是什么。

```cpp
int a=1;
cout<<typeid(a).name()<<endl;  //输出int

auto add_num1_and_num2 = [](int num1, int num2) -> int {
            return num1 + num2;
        };
cout<<typeid(add_num1_and_num2).name()<<endl;  //输出g5mainEUlvE_213，而且每次运行都不一样
```

其实，每个lambda表达式对象都是一个匿名类的实例，类型肯定是变化，但是，表达式的入参、返回值什么的都是固定，还是有办法规定类型的，其实就是通过模板类 function类  生成的匿名类。

## 3.1 使用泛型接收lambda表达式对象

```cpp
#include <iostream>
#include <string>
 
template <typename F>
void print(F const &f){
    std::cout<<f()<<std::endl;
}
 
int main() {
    std::cout << "Hello, World!" << std::endl;
 
    int num = 101;
    auto a = [&]//以引用的方式捕获本函数中的变量
             () //无参数
             ->std::string {//返回值的类型为std::string
        return std::to_string(num);
    };
    print(a);
    num++;
    print(a);
 
    return 0;
}
```

缺点很明显也无法避免，print函数的使用者可以传递任何类型的参数，如果传递有参数列表的lambda表达式对象，比如[](int a,int b){}，就要报错了，因为入参个数不同。

这种方式只能勉强，实际生产编码的时候根本不会这样用，

## 3.2 使用function模板类接收lambda表达式对象
```cpp
#include <iostream>
#include <functional>
#include <string>
 
void print(std::function<std::string ()> const &f){
    std::cout<<f()<<std::endl;
}
 
int main() {
    std::cout << "Hello, World!" << std::endl;
 
    int num = 101;
    auto a = [&]//以引用的方式捕获本函数中的变量
             () //无参数
             ->std::string {//返回值的类型为std::string
        return std::to_string(num);
    };
    print(a);
    num++;
    print(a);
 
    return 0;
}
```

std::function是个模板类；模板参数：返回值类型+函数参数类型列表；例如std::function<int  (int, int)> f，f可以指向任意一个返回指为int，参数为两个int类型的函数。

观察上面的程序，lambda表达式无参数，返回值类型为string；故在print函数中声明一个类型为function<std::string ()>类型的变量f，就可以用来接收a。注意：f前面必须加上const，否则编译过不了,在编译期就**避免偷摸着修改传递进来的lambda表达式对象**。至于加不加&，需要视程序而定，一般加上。

## 3.3 lambda作为入参的多线程实践
```cpp
#include <iostream>
#include <string>
#include <thread>
#include <chrono>
#include <vector>

int main() {
    std::vector<std::thread> _threads;

    // 新建20个线程，每个线程打印i
    for (int i = 0; i < 20; i++) {
        //定义一个lambda表达式a
        auto a = [=]() -> std::string {
            std::this_thread::sleep_for(std::chrono::milliseconds(i * 20));//休眠i*2ms
            return std::to_string(i);
        };

        std::cout << "when i is:" << i << "," << "address of lambda a:" << &a << std::endl;

        // 再定义一个wrapper包裹a
        auto wrapper = [&]() {
            //这里有其他操作
            std::string str = a();
            //这里有其他操作
            std::cout << str << "\t";
        };

        // 新建子线程，并把线程放到vector里面
        _threads.emplace_back(std::thread(wrapper));
    }

    //对所有线程调用join函数，避免主线程先于子线程退出。
    for (auto &t : _threads) {
        if (t.joinable()) {
            t.join();
        }
    }
    std::cout << std::endl;

    return 0;
}
```

输出：
```
when i is:0,address of lambda a:0x22fdec
when i is:1,address of lambda a:0x22fdec
when i is:2,address of lambda a:0x22fdec
when i is:3,address of lambda a:0x22fdec
when i is:4,address of lambda a:0x22fdec
when i is:5,address of lambda a:0x22fdec
when i is:6,address of lambda a:0x22fdec
when i is:7,address of lambda a:0x22fdec
when i is:8,address of lambda a:0x22fdec
8       when i is:9,address of lambda a:0x22fdec
when i is:10,address of lambda a:0x22fdec
when i is:11,address of lambda a:0x22fdec
when i is:12,address of lambda a:0x22fdec
when i is:13,address of lambda a:0x22fdec
when i is:14,address of lambda a:0x22fdec
when i is:15,address of lambda a:0x22fdec
15      when i is:16,address of lambda a:0x22fdec
when i is:17,address of lambda a:0x22fdec
17      when i is:18,address of lambda a:0x22fdec
when i is:19,address of lambda a:0x22fdec
19      19      19      19      19      19      19      19      19      19      19      19      19      19      19
19      19

Process finished with exit code 0
```

并没有按顺序输出0~19，原因：

1. auto a地址恰好都是同一个，且auto wrapper = [&]定义时，使用了引用捕获，导致在最终调用a时（在t.join()时发生调用），都是直接调用的最新的a对象
2. **有个问题，退出for循环后，i=19时产生的auto a这个对象，为啥还能被访问到啊？已经超出作用域了，也是在栈中的，竟然没被回收？？？？？？**超出作用域后，操作系统并不是直接将对象占用的栈空间完全抹除为低位，而是标记为可用，在那段栈空间恰好没有被覆写的情况下，是能通过原先的指针访问到的。


# 4 lambda表达式的本质
一个匿名类的、函数对象（Function<>对象）实例，这个函数对象实例是具名的，也就是lambda表达式的名字，这个对象重载了operator ()。

std::function，说白了，就是把函数对象化了。即，你可以把函数视作一个class的object只不过这个object有点特殊：它是一个可调用的对象（重载了operator() ）。为了实现函数对象化，**标准里加入了std::function，std::bind，还有lambda，还有mem_fn等**。

注意点：

* Lambda是表达式的一种，它是源码组成部分
* **闭包是Lambda表达式创建的运行期对象**，根据捕获方式的不同，它持有数据的副本或引用(也就是捕获的类型)
* 闭包类是实例化闭包的类。每个Lambda表达式都会触发编译器生成一个独一无二的闭包类，闭包中的语句为闭包类成员函数的可执行指令
* Lambda式常用于创建闭包并仅将其用作传递给函数的实参
* 闭包可以复制，对应于一个Lambda表达式的闭包类型可以有多个闭包，如下：

```cpp
int x;

auto c1 = [x](int y){ return x * y > 34; };

auto c2 = c1; // c2是c1的副本
auto c3 = c2; // c3是c2的副本
// c1，c2，c3都是同一Lambda式产生的闭包的副本
```

## 4.1 适用的场景

在C++11下，我们期望看到更多的泛型算法，而不希望看到以前的那种用函数去操作数据的代码了。由于我们现在甚至可以把函数都当作对象去处理了，所以这种期望成为了可能。比如，我们希望通过std::function等新语法，引导程序员写出这样的代码：

```cpp
array<string, 3> stringArr3 = {"ab", "def", "hello"};
auto func = [](const string & str1, const string & str2){ return str1.size() < str2.size(); };
sort(stringArr3.cbegin(), stringArr3.cend(), func);
```

而不希望看到你写个什么for循环然后再去排序。函数对象化的威力是强大的，你甚至可以把成员函数作为可调用的对象，**也就是Java里的方法引用，原理就是基于输入、输出的推断和生成的函数对象实例**。比如:

```cpp
find_if(stringArr3.cbegin(), stringArr3.cend(), mem_fn(&string::empty));
```

请注意：empty只是string类的非static成员函数。

std::function的执行时间代价很大，这很好理解，因为要根据函数生成一个可调用的class然后再实例化出一个对象然后再调用它。所以说，**到处都用std::function是一件很糟糕的事情**。


只有在使用泛型算法或者一些比较特殊的情况下才应该使用std::function，其它情况，比如即使要写callback（函数的参数是callback函数），那么除非你要保证这个函数可以接受任何形式的函数（包括可调用对象、lambda等），否则依然建议使用函数指针、模板这些传统的方法。


具体的一些用到的泛型算法和特场景举例（但是，很显然，能用lambda的场景，多打字也能用函数指针做到）：

* 标准库STL中使用，如std::find_if, std::remove_if, std::count_if等
* 自定义比较函数的算法，如std::sort, std::nth_element, std::lower_bound等
* 能够为std::unique_ptr/std::shared_ptr快速创建自定义析构器
* 临时创建回调函数、接口适配函数供一次性调用


## 4.2 lambda、Function对象、函数指针的相互转换

```cpp
int age = 1;

int &(*echo)(int &input);
echo = just_return;

using FUN_TYPE_PTR = int (*)(int);

auto fun = [&](int a) -> int {
    return a + age;
};

decltype(fun);

//没有捕获的lambda能赋值给  相应的函数指针  对象
FUN_TYPE_PTR aaaaaaa = fun;
std::function<int(int)> fun_b(fun);

//函数指针也能直接初始化函数对象
std::function<int(int)> fun_c(aaaaaaa);

//对象类型果然是不能随意转为C风格的东西
FUN_TYPE_PTR bbbbbb = fun_c;


std::cout << fun_b(5) << std::endl;

```


## 4.3 捕获的注意点：不要适用默认捕获
按引用或按值的默认捕获（就是[=]和[&]）都可能导致使用空悬的引用或者指针，导致程序未定义的行为！

先分别看一下，按引用和按值的默认捕获场景下的潜在问题

## 4.3.1 按引用默认捕获带来的问题


## 4.3.2 按值默认捕获带来的问题


## 4.3.3 C++14中的初始化捕获

c++14中不再支持默认捕获，而是改为初始化捕获（即前文使用过的广义Lambda捕获）。使用初始化捕获，有以下特点：

* 指定由Lambda式生成的闭包类中成员变量的名字
* 指定用以初始化成员变量的表达式
* 支持移动对象

```cpp
class Foo
{
public:
	bool isValidated() const;
}

auto func = [pf = std::make_unique_ptr<Foo>()] {return pf->isValidated();}; // 初始化捕获支持表达式
```

其中，捕获语句中：

* 等号左侧为闭包类的成员变量的名字，作用域为闭包类的作用域
* 等号右侧为初始化表达式，作用域与Lambda表达式定义处的作用域相同

与使用c11的Lambda式经常出现的问题相比，c14无疑是更好的选择。