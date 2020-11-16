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

# 2 待续
