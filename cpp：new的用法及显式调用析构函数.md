# 1 背景

参考文章：

C++中new的用法及显示调用析构函数  https://www.cnblogs.com/fnlingnzb-learner/p/9279039.html


new对象时加括号和不加括号时的差别  https://blog.csdn.net/itworld123/article/details/102731038

C++中使用placement new  https://blog.csdn.net/linuxheik/article/details/80449059


最近的在学习STL，看到分配器时，自定义的constructor和destroy用到了的奇怪的new语法和显式调用析构函数，学习记录一下。（正好看到参考文档的作者因为内存池的总结文章。）

# 2 cpp中new的用法
new是cpp中用于动态内存分配的运算符，配套的释放内存的关键字是delete/delete[]，在c中则是malloc/free。

cpp中的new的用法分为三类：
* 普通的new用法
* 无异常抛出的new用法
* 放置new

按照的以上顺序分别进行实践、解析。

## 2.1 plain new
plain new顾名思义就是普通的new，就是我们惯常使用的new。分配内存，调用构造函数，在C++中是这样定义的：

```cpp
void* operator new(std::size_t) throw(std::bad_alloc);
void operator delete(void *) throw();
```

plain new在分配失败的情况下，会直接抛出异常std::bad_alloc而不会返回nullptr，因此通过对plain new的返回值进行判空基本是无意义的（除非是事先声明为空指针，然后在catch try内进行内存分配,最后通过判空检查是否初始化成功）。比如：


```cpp
//
// Created by jeasconchan on 2020/9/19.
//
#include <iostream>
#include <limits>

namespace test_new {
    /**
     * 申请size长度的char数组，并返回首指针
     * @param size
     * @return
     */
    char *getMemory(unsigned long size) {
        return new char[size];
    }

}


int main() {
    {
        char *a = test_new::getMemory(666);
        std::cout << "index=666-1  " << a[666 - 1] << std::endl;
        std::cout << "index=666-2  " << a[666 - 2] << std::endl;
        //设置可以超过的数组的范围进行取值，只不过取出来的值可能因为类型对不上而报错
        //不过char占用1字节，基本是最小单位，按字节取时总能找到对应的asici值
        std::cout << "index=666+1 " << a[666 + 1] << std::endl;

        delete[] a;
    }

    {
        try {
            //申请分配无符号long类型的最大值个字节，显然我的内存没那么大，故意引发报错
            //注意！这里通过模板的方式取到每个类型的最大值
            char *a = test_new::getMemory(
                    std::numeric_limits<unsigned long>::max()
            );

            //上面new操作符抛异常就根本执行不到这一行判空
            if (nullptr == a) {
                std::cout << "char *a init failed!" << std::endl;
            }

        } catch (std::exception &exception) {
            std::cout << exception.what() << std::endl;
        }

    }


    return 0;
}
```


## 2.2 nothrow new
nothrow new是不抛出异常的运算符new的形式，并且，nothrow new在失败时，返回值为空指针


## 2.3 placement new
