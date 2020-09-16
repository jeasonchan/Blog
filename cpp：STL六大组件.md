# 1 背景

参考文档：

C语言中文网的系统介绍   http://c.biancheng.net/view/6593.html


STL 是由容器、算法、迭代器、函数对象、适配器、内存分配器这 6 部分构成，其中后面 4 部分是为前 2 部分服务的，即迭代器、函数对象、适配器、内存分配器这四个其实都是为了算法对容器中的数据进行操作。

容器：一些封装数据结构的模板类，例如 vector 向量容器、list 列表容器等。


算法：STL 提供了非常多（大约 100 个）的数据结构算法，它们都被设计成一个个的模板函数，这些算法在 std 命名空间中定义，其中大部分算法都包含在头文件 <algorithm> 中，少部分位于头文件 <numeric> 中。


迭代器：在 C++ STL 中，对容器中数据的读和写，是通过迭代器完成的，扮演着容器和算法之间的胶合剂。


函数对象：如果一个类将 () 运算符重载为成员函数，这个类就称为函数对象类，这个类的对象就是函数对象（又称仿函数）。


适配器：可以使一个类的接口（模板的参数）适配成用户指定的形式，从而让原本不能在一起工作的两个类工作在一起。值得一提的是，容器、迭代器和函数都有适配器。


内存分配器：为容器类模板提供自定义的内存申请和释放功能，由于往往只有高级用户才有改变内存分配策略的需求，因此内存分配器对于一般用户来说，并不常用。

 C++ 标准中，它们被组织为 13 个头文件:
 
 <iterator>、<functional>、<vector>、<deque>、<list>、<queue>、<stack>、<set>、<map>、<algorithm>、<numeric>、<memory>、<utility>

按照 C++ 标准库的规定，所有标准头文件都不再有扩展名，Pure CPP应该养成良好的习惯，遵照 C++ 规范，使用无扩展名的头文件。


## 1.1 STL容器是什么
STL 提供有 3 类标准容器，分别是序列容器、排序容器和哈希容器，其中后两类容器有时也统称为关联容器（关联式容器，默认从小到到大排序的是排序容器，C++11新增的基于哈希值的非排序关联容器，简称哈系容器）。

序列容器：主要包括 vector 向量容器、list 列表容器以及 deque 双端队列容器。之所以被称为序列容器，是因为元素**在容器中的位置同元素的值无关，即容器不是排序的**。将元素插入容器时，指定在什么位置，元素就会位于什么位置。

排序容器（排序关联式容器）：包括 set 集合容器、multiset多重集合容器、map映射容器以及 multimap 多重映射容器。**排序容器中的元素默认是由小到大排序好的，即便是插入元素，元素也会插入到适当位置。所以关联容器在查找时具有非常好的性能**。

哈希容器（非排序关联式容器）：C++ 11 刚刚新加入 4 种关联式容器，分别是 unordered_set 哈希集合、unordered_multiset 哈希多重集合、unordered_map 哈希映射以及 unordered_multimap 哈希多重映射。和排序容器不同，哈希容器中的元素是未排序的，元素的位置由哈希函数确定。

容器的存储方式不同，对增删改查的性能也不同，要根据的业务场景选择合适的数据容器。

## 1.2 STL迭代器是什么
http://c.biancheng.net/view/6675.html

无论是序列容器还是关联容器，最常做的操作无疑是遍历容器中存储的元素，而实现此操作，多数情况会选用“迭代器（iterator）”来实现。

不同容器的内部结构各异，但它们本质上都是用来存储大量数据的，诸如数据的排序、查找、求和等需要对数据进行遍历的操作方法应该是类似的。完全可以利用泛型技术，将它们设计成适用所有容器的通用算法，从而将容器和算法分离开。但实现此目的需要有一个类似中介的装置，它除了要具有对容器进行遍历读写数据的能力之外，还要能对外隐藏容器的内部差异，从而以统一的界面向算法传送数据。

STL 标准库为每一种标准容器定义了一种迭代器类型，这意味着，**不同容器的迭代器也不同，其功能强弱也有所不同**。

### 1.2.1 迭代器类别
常用的迭代器按功能强弱分为输入迭代器、输出迭代器、前向迭代器、双向迭代器、随机访问迭代器 5 种。本节主要介绍后面的这 3 种迭代器，即前向、双向、随机。


1. 前向迭代器（forward iterator）

假设 p 是一个前向迭代器，则 p 支持 ++p，p++，*p 操作，还可以被复制或赋值，可以用 == 和 != 运算符进行比较。此外，两个正向迭代器可以互相赋值。

2. 双向迭代器（bidirectional iterator）

双向迭代器具有正向迭代器的全部功能，除此之外，假设 p 是一个双向迭代器，则还可以进行 --p 或者 p-- 操作（即一次向后**移动一个位置**）。

3. 随机访问迭代器（random access iterator）

随机访问迭代器具有双向迭代器的全部功能。除此之外，假设 p 是一个随机访问迭代器，i 是一个整型变量或常量，则 p 还支持以下操作：

p+=i：使得 p 往后移动 i 个元素。

p-=i：使得 p 往前移动 i 个元素。

p+i：返回 p 后面第 i 个元素的迭代器。

p-i：返回 p 前面第 i 个元素的迭代器。

p[i]：返回 p 后面第 i 个元素的引用。

此外，两个随机访问迭代器 p1、p2 还可以用 <、>、<=、>= 运算符进行比较。另外，表达式 p2-p1 也是有定义的，其返回值表示 p2 所指向元素和 p1 所指向元素的序号之差（也可以说是 p2 和 p1 之间的元素个数减一）。

C++ 11 标准中不同容器指定使用的迭代器类型如下，正是因为各个数据结构支持的迭代器类型不同，所支持的算法也略有不同：

|容器	| 对应的迭代器类型 |
|---   |---|
| array	| 随机访问迭代器 |
| vector |	随机访问迭代器 |
| deque	| 随机访问迭代器 |
| list	| 双向迭代器 |
| set / multiset  |	双向迭代器 |
| map / multimap  | 双向迭代器 |
| forward_list | 前向迭代器  |
| unordered_map / unordered_multimap |	前向迭代器 |
| unordered_set / unordered_multiset |	前向迭代器 |
| stack	| 不支持迭代器 |
| queue	| 不支持迭代器 |

### 1.2.2 迭代器的定义方式
尽管不同容器对应着不同类别的迭代器，但这些迭代器有着较为统一的定义方式，使用\*迭代器名 表示迭代器指向的元素,因为：

```cpp
  template<typename _Tp, std::size_t _Nm>
    struct array
    {
      typedef _Tp 	    			      value_type;
      typedef value_type*          		      iterator;

      //......略
    };
```

可见iterato只是声明在array内部的一种类型，类型是指向_Tp的指针，因此，使用\*迭代器名 表示迭代器指向的元素

* 正向迭代器：	容器类名::iterator  迭代器名;
* 常量正向迭代器：	容器类名::const_iterator  迭代器名;
* 反向迭代器：	容器类名::reverse_iterator  迭代器名;
* 常量反向迭代器：	容器类名::const_reverse_iterator  迭代器名;

常量迭代器和非常量迭代器的分别在于，通过非常量迭代器还能修改其指向的元素。另外，反向迭代器和正向迭代器的区别在于：

* 对正向迭代器进行 ++ 操作时，迭代器会指向容器中的后一个元素；
* 而对反向迭代器进行 ++ 操作时，迭代器会指向容器中的前一个元素。

注意，以上 4 种定义迭代器的方式，并不是每个容器都适用。有一部分容器同时支持以上 4 种方式，比如 array、deque、vector；而有些容器只支持其中部分的定义方式，例如 forward_list 容器只支持定义正向迭代器，不支持定义反向迭代器。

```cpp
//
// Created by jeasconchan on 2020/9/13.
//

#include <array>
#include <iostream>
#include <vector>

int main() {
    {
        std::array<int, 10> intArray{};

        std::array<int, 10>::iterator iterator = intArray.begin();
        //auto iterator = intArray.begin();

        std::cout << iterator << std::endl;
    }

    {
        //遍历vector容器

        std::vector<int> intVector{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};

        std::cout << "第一种遍历方法：通过index" << std::endl;
        int size = intVector.size();

        for (int i = 0; i < size; ++i) {
            //像普通数组一样，通过重载的元算符[]进行访问
            std::cout << intVector[i] << " ";
        }
        std::cout << std::endl;


        std::cout << "第二种遍历方法：通过迭代器和迭代器不等于的比较" << std::endl;
        std::vector<int>::iterator normalIterator;

        //其实，不应该在上面定义，因为下面的=会发生新值覆盖旧值的赋值过程
        //不过iterator本质上就是指针，这种性能损耗忽略不计，而且是不可避免的
        //此处使用!=来进行迭代器比较
        //和java一样，end()是指向最后元素的下一个位置，一个为止类型的地址
        for (normalIterator = intVector.begin(); normalIterator != intVector.end(); ++normalIterator) {
            std::cout << *normalIterator << " ";
        }
        std::cout << std::endl;

        std::cout << "第三种遍历方法：通过迭代器和迭代器小于号的比较" << std::endl;
        for (normalIterator = intVector.begin(); normalIterator < intVector.end(); ++normalIterator) {
            std::cout << *normalIterator << " ";
        }
        std::cout << std::endl;

        std::cout << "第四种遍历方法：通过while循环间隔输出" << std::endl;
        normalIterator = intVector.begin();
        while (normalIterator < intVector.end()) {
            std::cout << *normalIterator << " ";
            normalIterator += 5;

            //跃界后，指向了不属于vector的地址，虽然可以读出int的值，但是其实是无意义的值
            std::cout << "地址是：" << &(*normalIterator) << " 值是：" << *normalIterator << std::endl;
        }


    }
    return 0;
}

```

list简单操作：

```cpp
 {
        //list的迭代器是双向迭代器
        list<int> intList{1, 2, 3, 4, 5};
        list<int>::iterator it;
        list<int>::iterator constIt;

        for (it = intList.begin(); it != intList.end(); ++it) {
            cout << *it << " ";
        }

        //直接编译不通过
        //因为list::iterator这个类型没有重载<运算符
        //for (it = intList.begin(); it < intList.end(); ++it) {
        //    cout << *it << " ";
        //}

        //同理，以下也编译不通过
        //for (int i = 0; i < intList.size(); ++i) {
        //    cout << intList[i];
        //}

}
```

# 2 STL序列式容器
所谓序列容器，即以线性排列来存储某一指定类型（例如 int、double 等）的数据，该类容器并不会自动对存储的元素按照值的大小进行排序。后面章节的STL关联式容器，会默认按照从小到的顺序排列，再后面还有无序关联式容器，是基于哈希值的所以是无序的。

* array<T,N>（数组容器）：表示可以存储 N 个 T 类型的元素，是 C++ 本身提供的一种容器。此类容器一旦建立，其长度就是固定不变的，这意味着不能增加或删除元素，只能改变某个元素的值；

* vector<T>（向量容器）：用来存放 T 类型的元素，是一个长度可变的序列容器，即在存储空间不足时，会自动申请更多的内存。使用此容器，在尾部增加或删除元素的效率最高（时间复杂度为 O(1) 常数阶），在其它位置插入或删除元素效率较差（时间复杂度为 O(n) 线性阶，其中 n 为容器中元素的个数）；

* deque<T>（双端队列容器）：和 vector 非常相似，区别在于使用该容器不仅尾部插入和删除元素高效，在头部插入或删除元素也同样高效，时间复杂度都是 O(1) 常数阶，但是在容器中某一位置处插入或删除元素，时间复杂度为 O(n) 线性阶；

* list<T>（链表容器）：是一个长度可变的、由 T 类型元素组成的序列，它以双向链表的形式组织元素，在这个序列的任何地方都可以高效地增加或删除元素（时间复杂度都为常数阶 O(1)），但访问容器中任意元素的速度要比前三种容器慢，这是因为 list<T> 必须从第一个元素或最后一个元素开始访问，需要沿着链表移动，直到到达想要的元素。

* forward_list<T>（正向链表容器）：和 list 容器非常类似，只不过它以单链表的形式组织元素，它内部的元素只能从第一个元素开始访问，是一类比链表容器快、更节省内存的容器。

**在逻辑组织存储形式和常见的成员函数见http://c.biancheng.net/view/409.html**


## 2.1 array用法详解
array 容器是 C++ 11 标准中新增的序列容器，简单地理解，它就是在 C++ 普通数组的基础上，添加了一些成员函数和全局函数。在使用上，它比普通数组更安全（比如，通过index访问有越界检查），且效率并没有因此变差。

```cpp
{
    constexpr int num = 10;

    //疑问，第二个模板参数如何限制为常量的？？？
    array<int, num> intArray{0, 1, 2, 3};

    //对于array，以下两个成员函数等效
    cout << intArray.size() << endl;
    cout << intArray.max_size() << endl;

    try {
        cout << intArray.at(100) << endl;
    } catch (exception &e) {
        cerr << e.what();
    }
    cout << intArray.at(3) << endl;

    //front()返回值是预案是元素的引用
    cout << intArray.front() << endl;

    //以下三个函数的返回值都是都是指针
    cout << intArray.begin() << endl;
    cout << intArray.data() << endl;
    cout << *intArray.data() << endl;

    //标准库提供的全局函数begin，对于array,其实就是会调用容器的begin成员函数
    //对于普通数组，返回第一个元素的指针
    std::begin(intArray);

    //重载全局的get方法，适配入参是array的情况
    std::get<3>(intArray);

}
```

## 2.2 array随机访问迭代器详解


# 3 STL关联式容器（有序的）


# 3 STL无序关联式容器


# 4 STL容器适配器

```cpp
{
    //CPP适配器，利用木板技术服用已存在的函数的设计模式
    //比如，stack，底层会默认复用双端队列，也就是有个成员函数就是双端队列
    stack<int> intStack{};
    intStack.push(1);
}

```

# 5 STL迭代器适配器



# 6 STL分配器
allocator（分配器）是CPP标准库的重要组成部分。对于大小可以动态变化的容器，对其内存空间的管理十分重要，也就是动态分配内存的要求很高，于是出现了分配器，用于管理对内存的分配和释放。默认情况下，CPP标准库使用自带的通用分配器，同时，程序员也可以根据需求自定义分配器。


STL在引入CPP标准时，分配器的可定制程度非常高，给STL带来了很高的抽象能力，但是，高抽象也会带来一部分损耗，因此CPP标准委员会对分配器进行了限制，以减少性能损耗。所以，目前的分配器比一开始的hp版分配器有较大的定制性削弱。


分配器一般用处：

1. 封装对不同类型的内存空间的访问方式

2. 实现内存池（和线程池类似的原理），提高内存分配的性能

假设class A在自身的生命周期内需要对T进行内存管理，则需要在A的域内提供对T相关类型的定义：

```cpp
namespace jeason {
    template<typename T>
    class A {
    public:
        using value_type = T;

        using pointer = T *;
        using const_pointer = const T *;

        using reference = T &;
        using const_reference = const T &;

        //所用内存大小的类型，表示A所定义的分配模型中的单个对象最大尺寸的无符号整型
        //STL中都是直接用的std::size_t
        using size_type = std::size_t;

        //指针差值的类型，为有符号整形，用于表示分配模型内的两个指针的差值
        //STL中都是直接用的std::ptrdiff_t
        using difference_type = std::ptrdiff_t;




    };


}
```

比如，在CPP的c++/8/bits/allocator.h头文件中的一个分配器源码：

```cpp
  /// allocator<void> specialization.
  template<>
    class allocator<void>
    {
    public:
      typedef size_t      size_type;
      typedef ptrdiff_t   difference_type;
      typedef void*       pointer;
      typedef const void* const_pointer;
      typedef void        value_type;

      template<typename _Tp1>
	struct rebind
	{ typedef allocator<_Tp1> other; };

#if __cplusplus >= 201103L
      // _GLIBCXX_RESOLVE_LIB_DEFECTS
      // 2103. std::allocator propagate_on_container_move_assignment
      typedef true_type propagate_on_container_move_assignment;

      typedef true_type is_always_equal;

      template<typename _Up, typename... _Args>
	void
	construct(_Up* __p, _Args&&... __args)
	{ ::new((void *)__p) _Up(std::forward<_Args>(__args)...); }

      template<typename _Up>
	void
	destroy(_Up* __p) { __p->~_Up(); }
#endif
    };
```
疑问：

construct和destroy里面是什么语法？？？？




# 7 CPP常用算法
