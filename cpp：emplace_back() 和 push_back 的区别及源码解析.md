# 1 背景

参考文档：https://blog.csdn.net/xiaolewennofollow/article/details/52559364

empalce_back源码解析   https://www.itdaan.com/tw/54b3594ddcfb1ea5abd60027d3f9c295

leetcode刷题时用到了std::vector，想到了新增的emplace_back() 方法，好奇和老的push_back有什么区别，在此记录一下。

同时在实验的过程中，发现emplace_back好像能触发多个入参的构造函数类型转换，并且explict无法限制emplace_push的这一隐式类型转换，只能限制push_back的隐式类型转换，并且，push_back之支持当个入参的隐式转换，十分神奇。经过查看源码，结论是：cpp语言级别的隐式转换是特指利用只有一个入参构造函数进行转换，emplace_back的转换实际上是通过分配器实现的，将这些入参转发给构造函数进行对象的初始化。

# 2 区别
在引入右值引用、转移构造函数、转移赋值运算符之前，在使用push_back()向容器中加入一个右值元素（临时对象）的时候，首先会调用拷贝构造函数利用这个临时对象来初始化形参，然后需要调用拷贝构造函数将这个初始化万的形参对象放入容器空间中。从这个过程来看，显然有一份对象空间是浪费的，完全可以节省。

因此，引入了右值引用、转移构造函数后，push_back()的入参是右值时就会调用转移构造函数来初始化形参，再用转移构造函数来初始化容器中的空间。但是，中间还是夹杂了一个多余的空间，有没有可能一步到位直接用入参去初始化容器空间中的内存？？？

当然可以，那就是emplace_back，也就是底层利用 放置new 实现，直接用入参初始化容器中的对象空间。

## 2.1 原原型和代码示例
我使用的centos的源码是：
```cpp
#if __cplusplus >= 201103L
      template<typename... _Args>
#if __cplusplus > 201402L
	reference
#else
	void
#endif
	emplace_back(_Args&&... __args)
	{
	  push_back(bool(__args...));
#if __cplusplus > 201402L
	  return back();
#endif
	}
```
当编译条件是cpp17时，函数原型就是：
```cpp
template<typename... _Args>
reference	emplace_back(_Args&&... __args) {
	  push_back(bool(__args...));
	  return back();
}
```
也就是各模板函数，看上去底层还是调用了push_back，也就是，我看到的这个实现，应该是push_back已经使用 放置new  重新实现了………

现在卡来实践一下emplace_back是不是真的减少了一次对象构造：

```cpp
#include <vector>  
#include <string>  
#include <iostream>  

struct President  
{  
    std::string name;  
    std::string country;  
    int year;  

    President(std::string p_name, std::string p_country, int p_year)  
        : name(std::move(p_name)), country(std::move(p_country)), year(p_year)  
    {  
        std::cout << "I am being constructed.\n";  
    }
    President(const President& other)
        : name(std::move(other.name)), country(std::move(other.country)), year(other.year)
    {
        std::cout << "I am being copy constructed.\n";
    }
    President(President&& other)  
        : name(std::move(other.name)), country(std::move(other.country)), year(other.year)  
    {  
        std::cout << "I am being moved.\n";  
    }  
    President& operator=(const President& other);  
};  

int main()  
{  
    std::vector<President> elections;  
    std::cout << "emplace_back:\n";  
    
    //按道理，这一表达式应该是只发生一次对象构造
    elections.emplace_back("Nelson Mandela", "South Africa", 1994); 

    std::vector<President> reElections;  
    std::cout << "\npush_back:\n";  
    reElections.push_back(President("Franklin Delano Roosevelt", "the USA", 1936));  

    std::cout << "\nContents:\n";  
    for (President const& president: elections) {  
       std::cout << president.name << " was elected president of "  
            << president.country << " in " << president.year << ".\n";  
    }  
    for (President const& president: reElections) {  
        std::cout << president.name << " was re-elected president of "  
            << president.country << " in " << president.year << ".\n";  
    }

}
```

输出：
```
emplace_back:
I am being constructed.//果然，放置new，只发生了一次对象构建

push_back:
I am being constructed.
I am being moved.

Contents:
Nelson Mandela was elected president of South Africa in 1994.
```

emplace_back的实现原理可以简单的理解为：

```cpp
//直接给vector申请对象空间，并直接在申请好的空间上进行初始化
Student* ptr = (Student*)new char[sizeof(Student)];
new (ptr)Student(10);
```

# 3 多入参的"隐式转换"实现原理
重点：分配器、可变长模板参数

