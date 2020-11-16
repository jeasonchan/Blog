# 1 背景
最近在研究一个cpp的json库，看到一个很有趣的用法：

```cpp
// create object from string literal
json j = "{ \"happy\": true, \"pi\": 3.141 }"_json;
```
也就是，_json充当了一个标记的功能，编辑这个字符串字面其实是可以变为json实例。

源码是这样的：
```cpp
/*!
@brief user-defined string literal for JSON values

This operator implements a user-defined string literal for JSON objects. It
can be used by adding `"_json"` to a string literal and returns a JSON object
if no parse error occurred.

@param[in] s  a string representation of a JSON object
@param[in] n  the length of string @a s
@return a JSON object

@since version 1.0.0
*/
JSON_HEDLEY_NON_NULL(1)
inline nlohmann::json operator "" _json(const char* s, std::size_t n)
{
    return nlohmann::json::parse(s, s + n);
}
```

也就是，_json其实是CPP的标准：user-defined string literal，也就是字符串字面。在此记录学习一下字符串字面量：

参考文档：

微软CPP手册  https://docs.microsoft.com/en-us/cpp/cpp/user-defined-literals-cpp?view=msvc-160

CPP官方手册   https://zh.cppreference.com/w/cpp/language/user_literal

# 2 什么是用户自定义字面
C + + 中有六种主要类别的字面量：整数、字符、浮点、字符串、布尔值和指针。 从 C + + 11 开始，可以根据这些类别定义自己的字面量，为常见惯例提供**句法快捷方式并提高类型安全性**。 例如，假设有一个 Distance 类,为了方便字面两量直接转为的公里类或者英里类的实例，可以在定义 用户自定义字面量，然后这样使用：
```cpp
auto d = 42.0_km;
auto d = 42.0_mi;
```
The Standard Library has user-defined literals for std::string, for std::complex, and for units in time and duration operations in the <chrono> header:
标准库已经包含了很多用户自定义字面两，比如：std::string的字面量, for std::complex的字面量，<chrono>头中为时间点和时间段定义的字面量单位，使用方式如下：
    
```cpp
Distance d = 36.0_mi + 42.0_km;         // Custom UDL (see below)，36.0_mi根据_mi的定义直接转成一个对象
std::string str = "hello"s + "World"s;  // Standard Library <string> UDL
complex<double> num = (2.0 + 3.01i) * (5.0 + 4.3i);        // Standard Library <complex> UDL
auto duration = 15ms + 42h;             // Standard Library <chrono> UDLs
```
在本文中，没啥修饰的字面量，称之标准字面量；使用自定义字面量符号修饰的字面量，称之为 自定义字面量，自定义字面量的值 究竟是什么，要看这个 字面量操作符  的声明、定义。


# 3 用户自定字面量 操作符 声明
声明、实现用户自定义字面量操作符，必须使用下面的形式（用户自定义字面量本质上就是操作符、函数）之一：
```cpp
ReturnType operator "" _a(unsigned long long int);   // Literal operator for user-defined INTEGRAL literal
ReturnType operator "" _b(long double);              // Literal operator for user-defined FLOATING literal
ReturnType operator "" _c(char);                     // Literal operator for user-defined CHARACTER literal
ReturnType operator "" _d(wchar_t);                  // Literal operator for user-defined CHARACTER literal
ReturnType operator "" _e(char16_t);                 // Literal operator for user-defined CHARACTER literal
ReturnType operator "" _f(char32_t);                 // Literal operator for user-defined CHARACTER literal
ReturnType operator "" _g(const char*, size_t);      // Literal operator for user-defined STRING literal
ReturnType operator "" _h(const wchar_t*, size_t);   // Literal operator for user-defined STRING literal
ReturnType operator "" _i(const char16_t*, size_t);  // Literal operator for user-defined STRING literal
ReturnType operator "" _g(const char32_t*, size_t);  // Literal operator for user-defined STRING literal
ReturnType operator "" _r(const char*);              // Raw literal operator
template<char...> ReturnType operator "" _t();       // Literal operator template
```
也就是：

1. operator和"函数名"/操作符名称 之间有个双引号
2. 操作符号名名称必须使用下划线开头，因为，只允许标准库能定义不带下划线开头的 用户自定义字面量
3. 返回值根据自己的需求来定义，同时，自己确认经过自定义字面量转换过的返回值是常量的情况下，可以用constexpr修饰返回值

# 4 转换字面量
在CPP源码中，任何字面量，包括我们的自定义字面量，肯定是 标准字面量+字面量操作符（如果有的话） 这种展现模式，对于标准的字面量，编译器能准确的将他们分别识别为integer, float, const char\* string 等等。对于自定义字面量，因为编译器需要处理才能返回这些自定义字面量真实的值，因此自定义字面量被称为Cooked literals，除了_r and _t这两个字面量操作符，因为，_r就是不处理的意思。

比如，一个这样的字面量42.0_km将通过字面量操作符operator "" _km()完成转换，42_a将通过上面提到的 ReturnType operator "" _a(unsigned long long int); 完成转换。

下面的例子展示了通过自定义字面量，能大大减少了代码亮，看上去会非常简洁：
```cpp
// UDL_Distance.cpp

#include <iostream>
#include <string>

struct Distance
{
private:
    explicit Distance(long double val) : kilometers(val)
    {}

    friend Distance operator"" _km(long double val);
    friend Distance operator"" _mi(long double val);

    long double kilometers{ 0 };
public:
    const static long double km_per_mile;
    long double get_kilometers() { return kilometers; }

    Distance operator+(Distance other)
    {
        return Distance(get_kilometers() + other.get_kilometers());
    }
};

const long double Distance::km_per_mile = 1.609344L;

Distance operator"" _km(long double val)
{
    return Distance(val);
}

Distance operator"" _mi(long double val)
{
    return Distance(val * Distance::km_per_mile);
}

int main()
{
    // Must have a decimal point to bind to the operator we defined!
    Distance d{ 402.0_km }; // construct using kilometers
    std::cout << "Kilometers in d: " << d.get_kilometers() << std::endl; // 402

    Distance d2{ 402.0_mi }; // construct using miles
    std::cout << "Kilometers in d2: " << d2.get_kilometers() << std::endl;  //646.956

    // add distances constructed with different units
    Distance d3 = 36.0_mi + 42.0_km;
    std::cout << "d3 value = " << d3.get_kilometers() << std::endl; // 99.9364

    // Distance d4(90.0); // error constructor not accessible

    std::string s;
    std::getline(std::cin, s);
    return 0;
}
```

上面的代码中，有几个注意点：

1. 自定义字面量操作符声明、实现中，返回值都直接是值，充分信任了编译器的RVO，如果要在语言层面实现返回值优化，要借助  转移语义  或者  智能指针  实现资源转移。（不过这个Distance也只有一成员变量，不涉及大规模资源拷贝）。

2. 使用了纯粹的CPP初始化方式花括号初始化对象，long double kilometers{ 0 }，且在声明对象成员时直接设置了默认值，和java中的思路一致
