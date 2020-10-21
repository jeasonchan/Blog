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

再来看一下操作系统的里面emplace_back的源码：

```cpp
public:
template<class... _Valty>
decltype(auto) emplace_back(_Valty&&... _Val)
{   // insert by perfectly forwarding into element at end, provide strong guarantee
if (_Has_unused_capacity())
    {
    _Emplace_back_with_unused_capacity(_STD forward<_Valty>(_Val)...);
    }
else
    {   // reallocate
    const size_type _Oldsize = size();

    if (_Oldsize == max_size())
	{
	_Xlength();
	}

    const size_type _Newsize = _Oldsize + 1;
    const size_type _Newcapacity = _Calculate_growth(_Newsize);
    bool _Emplaced = false;
    const pointer _Newvec = this->_Getal().allocate(_Newcapacity);
    _Alty& _Al = this->_Getal();

    _TRY_BEGIN
    _Alty_traits::construct(_Al, _Unfancy(_Newvec + _Oldsize), _STD forward<_Valty>(_Val)...);
    _Emplaced = true;
    _Umove_if_noexcept(this->_Myfirst(), this->_Mylast(), _Newvec);
    _CATCH_ALL
    if (_Emplaced)
	{
	_Alty_traits::destroy(_Al, _Unfancy(_Newvec + _Oldsize));
	}

    _Al.deallocate(_Newvec, _Newcapacity);
    _RERAISE;
    _CATCH_END

    _Change_array(_Newvec, _Newsize, _Newcapacity);
    }

#if _HAS_CXX17
        return (this->_Mylast()[-1]);
#endif /* _HAS_CXX17 */
}
```

该版本的源码和第二章节的源码在大体上是一致的，只不过第二章中抽了一个方法。接下看一下代码中的细节。

看了整段代码，将对象放入队列是分两种情况秒，一种是size<capacity，一种是size==capacity，相等时就要扩容了，扩容之后，就要往队列里放新对象了，构造新对象必然用到入入参的_Val，然后用到这个的只有这一行：
```cpp
 _Alty_traits::construct(_Al, _Unfancy(_Newvec + _Oldsize), _STD forward<_Valty>(_Val)...);
```
便通过 _Alty_traits::construct来创建对象,并最终利用强制类似装换的指针来指向容器类之中对应类的构造函数, 并且利用可变长模板 将构造函数所需要的内容传递过去构造新的对象。而该函数的实现如下：

```cpp
template<class _Objty,
class... _Types>
static void construct(_Alloc&, _Objty * const _Ptr, _Types&&... _Args)
{   // construct _Objty(_Types...) at _Ptr
::new (const_cast<void *>(static_cast<const volatile void *>(_Ptr)))
    _Objty(_STD forward<_Types>(_Args)...);
}
```
这里最为巧妙的部分就是利用可变长模板实现了，任意传参的对象构造。可变长模板是C++11新引进的特性，接下来我们来详细看看可变长模板是如何来使用，来实现任意长度的参数呢?

## 3.1 可变长模板函数的定义及实现

```cpp
template <class... T>
    void f(T... args);
```
通过template来声明参数包args，这个参数包中可以包含0到任意个参数，并且作为函数参数调用。之后我们便可以在函数之中将参数包展开成一个一个独立的参数。

通过不断递归的方式，提取可变长模板参数之中的首个元素，并且设置递归的终止点的方式来依次处理各个元素。这种处理函数的方式本质上就是在通过递归的方式处理列表，这种编程思路在函数式编程语言之中十分常见,在C++之中看到这样的用法，也让笔者作为C++的入门选手感到很新奇。笔者曾经接触过Scala与Erlang语言之中大量利用了这种写法，但是多层递归导致的必然是栈调用的开销变大，利用尾递归的方式来优化这样的写法，才能减少非必要的函数调用开销。

比如下面实现的求一组输入数据的最大值：
```cpp
template<typename t1,typename ...t2> t1 max_num(t1 num, t2 ...args) {
    auto n = max_num(args...);
    return n > num ? n : num;
}
template<typename t1> t1 max_num(t1 num) {
    return num;
}
```
