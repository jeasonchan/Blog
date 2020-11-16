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
C + + 中有六种主要类别的字面量：整数、字符、浮点、字符串、布尔值和指针。 从 C + + 11 开始，可以根据这些类别定义自己的字面量，为常见惯例提供**句法快捷方式并提高类型安全性**。 例如，假设有一个 Distance 类,为了方便字面两直接转为的公里类后者英里类，可以在使用用户自定义字面两，然后这样使用：
```cpp
auto d = 42.0_km;
auto d = 42.0_mi
```
The Standard Library has user-defined literals for std::string, for std::complex, and for units in time and duration operations in the <chrono> header:
标准库已经包含了很多用户自定义字面两，比如：std::string的字面量, for std::complex的字面量，<chrono>头中为时间点和时间段定义的字面量单位，使用方式如下：
    
```cpp
Distance d = 36.0_mi + 42.0_km;         // Custom UDL (see below)，36.0_mi根据_mi的定义直接转成一个对象
std::string str = "hello"s + "World"s;  // Standard Library <string> UDL
complex<double> num = (2.0 + 3.01i) * (5.0 + 4.3i);        // Standard Library <complex> UDL
auto duration = 15ms + 42h;             // Standard Library <chrono> UDLs
```
