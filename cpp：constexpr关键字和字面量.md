# 0 背景
使用过python里的tuple之后的，温习下的CPP的tuple：

```cpp
auto myTuplple = std::make_tuple(1, 2);


// 好奇这个工具函数的实现，于是看了一下：
constexpr tuple<typename __decay_and_strip<_Elements>::__type...>
make_tuple(_Elements&&... __args)
{
    typedef tuple<typename __decay_and_strip<_Elements>::__type...>
__result_type;
    return __result_type(std::forward<_Elements>(__args)...);
}

```
函数声明和返回值，看上去都好像是值传递，STL不担心由性能问题吗？还是constexpr有什么特殊作用？


参考文档：

微软关于字面量文档  https://docs.microsoft.com/en-us/cpp/cpp/trivial-standard-layout-and-pod-types?view=msvc-160

微软关于contexpr的文档  https://docs.microsoft.com/en-us/cpp/cpp/constexpr-cpp?view=msvc-160

constexpr关键字介绍  https://en.cppreference.com/w/cpp/language/constexpr

constexpr if  结构   https://en.cppreference.com/w/cpp/language/if



# 1 字面量
A literal type is one whose layout（内存布局） can be determined at compile time. The following are the literal types:

* void类型，不多说
* scalar types，标量类型，一般都直接放在静态区了
* references，引用类型，包括左值引用和右值引用
* Arrays of void, scalar types or references，上面类的C数组
* A class that has a trivial destructor（普通的析构函数，不需要做额外动作的，比如，手动delete成员指针等等）, and one or more constexpr constructors that are not move or copy constructors. Additionally, all its non-static data members and base classes must be literal types and not volatile.


**其实，只要记住，能在编译期就完全确定内存分布的"东西"就是字面量，这就要求不能用到堆内存。所以，如果父类或者本身的成员变量，只要有指向堆的指针，注定不符合字面量要求了。**


在代码中判断的一个的"东西"是不是字面量可直接使用元编程经常用到type_traits头文件中的类模板  std::is_literal_type ，还有其他类似的类模板， 比如：is_const、is_trivial、is_reference


# 2 constexpr
这个关键字既可以修饰变量也可以修饰函数。

## 2.1 用来修饰变量
很简单，出了和const相同的定义就初始化之外，还多一个要求，constexpr变量必须能够在编译期就得出一个确定的值，这就要求我们，要么使用字面量，要么使用常量表达式了。

```cpp
constexpr literal-type identifier = constant-expression ;
constexpr literal-type identifier { constant-expression } ;
constexpr literal-type identifier ( params ) ;
constexpr ctor ( params ) ;
```

## 2.2 用来修饰函数
A constexpr function is one whose return value is computable at compile time **when consuming code requires it.**

所以，constexpr函数是具有在编译期就返回一个常量的能力的，能够用来初始化一个constexpr变量。


# 2 结论
总的来说，constexpr只是声明变量和函数**有可能**在编译器算出一个常量值出来，同时也说明了，在运行期，我们同样可以的调用constexpr函数，并在运行期得出一个值来。


