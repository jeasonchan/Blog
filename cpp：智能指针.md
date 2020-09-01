# 1 背景
参考文章：


各包装类的接口简单实践    http://www.neohope.org/2018/08/15/%E6%B5%85%E8%B0%88cpp%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88/


# 2 接口简单实践

```cpp
//添加cpp的头文件，c的是memory.h
#include <memory>
#include <iostream>
#include <string>

using namespace std;

namespace jeason {
    class Test {
    public:
        Test(int age, string &&name) :
                age(age), name(name) {
            cout << "右值引用有参数构造" << endl;
        }

        Test(int age, string &name) :
                age(age), name(name) {
            cout << "左值引用有参数构造" << endl;
        }

        /*
         * 无餐构造
         */
        Test() {
            cout << "无参构造" << endl;
        }

        Test(Test &test) {
            cout << "复制拷贝构造" << endl;
        }

        Test(Test &&test) noexcept {
            cout << "移动拷贝构造" << endl;
        }

        ~Test() {
            cout << "析构函数" << endl;
        }

        void printDetail() {
            cout << "name:" << this->name << endl
                 << "age:" << this->age << endl;
        }

        int age;
        string name;

    };


    void test_auto_ptr() {
        //这个范型类已经过时
        string name = "jeason";

        //不能直接想下面的这么用
        //auto_ptr<Test> autoPtrTest(new Test(1, name));
        //因为，"123"无法向string &进行隐式转换，
        //因为string &name="123"这句话是非法的

        auto_ptr<Test> autoPtr_1(new Test(1, name));

        if (autoPtr_1.get()) {
            autoPtr_1->printDetail();

            autoPtr_1->name = "new name";
            autoPtr_1->printDetail();

            (*autoPtr_1).name = "new name 2";
            autoPtr_1->printDetail();
        }

        /*
         用旧值去初始化一个新对象，完全等价于
         auto_ptr<Test> autoPtr_2(autoPtr_1);
         也就是会调用左值拷贝构造函数,看了一下拷贝函数的实现，

         auto_ptr(auto_ptr& __a) throw() : _M_ptr(__a.release()) { }

         也就是autoPtr_1持有的对象指针被置为null，被析构时持有的对象不会被释放，
         因为地址被转移到新对象中
         */
        auto_ptr<Test> autoPtr_2 = autoPtr_1;

        if (nullptr == autoPtr_1.get()) {
            cout << "autoPtr_1不再持有对象的指针" << endl;
        }

        //其实，写成autoPtr_1.reset(nullptr);可读性更好
        autoPtr_2.reset();

        if (nullptr == autoPtr_2.get()) {
            cout << "autoPtr_2不再持有对象的指针" << endl;
        }


    }

    void test_unique_ptr() {
        //独占对象
        //保证指针所指对象生命周期与其一致
        unique_ptr<Test> unique_ptr_01(new Test(20, "tom"));
        unique_ptr_01->printDetail();

        /*
        不允许使用这两个函数：
         Disable copy from lvalue.
        unique_ptr(const unique_ptr&) = delete;
        unique_ptr& operator=(const unique_ptr&) = delete;
         */
        //unique_ptr<Test> unique_ptr_02 = unique_ptr_01;

        //需要通过move来处理，间接调用右值引用拷贝构造函数，实现指针转移
        unique_ptr<Test> unique_ptr_03 = move(unique_ptr_01);
        if (!unique_ptr_01)cout << "unique_ptr_01 is empty" << endl;
        unique_ptr_03->printDetail();

        //释放指针
        unique_ptr_03.reset();
        if (!unique_ptr_03)cout << "unique_ptr_03 is empty" << endl;

    }


    void test_share_ptr() {
        shared_ptr<Test> shared_ptr_01(make_shared<Test>(20, "tom"));
        shared_ptr<Test> shared_ptr_02 = shared_ptr_01;
        shared_ptr_01->printDetail();
        shared_ptr_02->printDetail();

        shared_ptr_01.reset();
        if (!shared_ptr_01)cout << "shared_ptr_01 is empty" << endl;
        shared_ptr_02->printDetail();

        shared_ptr_02.reset();
        if (!shared_ptr_02)cout << "shared_ptr_02 is empty" << endl;

    }


    void test_weak_ptr() {
        shared_ptr<Test> shared_ptr_01(make_shared<Test>(20, "tom"));
        weak_ptr<Test> weak_ptr_01 = shared_ptr_01;
        shared_ptr_01->printDetail();
        weak_ptr_01.lock()->printDetail();

        weak_ptr_01.reset();
        if (!weak_ptr_01.lock())cout << "weak_ptr_01 is empty" << endl;
        shared_ptr_01->printDetail();

        weak_ptr<Test> weak_ptr_02 = shared_ptr_01;
        weak_ptr<Test> weak_ptr_03 = weak_ptr_02;
        if (weak_ptr_01.lock())weak_ptr_02.lock()->printDetail();
        shared_ptr_01.reset();
        if (!weak_ptr_01.lock())cout << "weak_ptr_02 is empty" << endl;

    }


}


int main() {

    jeason::test_auto_ptr();

    cout << "=================================" << endl;

    jeason::test_share_ptr();

    cout << "=================================" << endl;

    jeason::test_unique_ptr();

    cout << "=================================" << endl;

    jeason::test_weak_ptr();


    return 0;
}
```

通过以上实践发现，CPP的智能指针，与JVM内存中的四种引用方式，强引用、软引用、弱引用，虚引用，有很多相似的地方。

通过以上对过时的auto_ptr和正在使用时的share、unique、weak指针的简单操作和简单的源码阅读，知道了个皮毛，下面开始稍微深入介绍。

# 3 智能指针

C++中的三种智能指针分析（RAII思想）     https://blog.csdn.net/GangStudyIT/article/details/80645399

【C++】动态内存管理和智能指针    https://blog.csdn.net/bit_clearoff/article/details/53861627?utm_source=blogxgwz6


首先我们在理解智能指针之前我们先了解一下什么是RAII思想。RAII（Resource Acquisition Is Initialization）机制是Bjarne Stroustrup首先提出的，是一种利用对象生命周期来控制程序资源（如内存、文件句柄、网络连接、互斥量等等）的简单技术。

对于RAII概念清楚后，我们就可以理解为智能指针就是RAII的一种体现，智能指针呢，它是利用了类的构造和析构，用一个类来管理资源的申请和释放，也就是说，**存储对象的堆内存就是我们要通过RAII理念进行管理的资源**。


## 3.1 为什么要有智能指针
```cpp
void Fun()
{
    int *p = new int[1000];
    throw int(); //异常的抛出
    delete[] p;
}

int main()
{
    try
    {
        Fun();
    }
    catch (exception e)
    {
        printf("异常\n"); // 捕捉
    }
    return 0;
}
```

呵呵，原博主上面的举例代码，异常处理根本无法捕捉到的抛出的int……**真的傻逼**……

如果在运行中我们开出了1000个int字节的空间，但是在我们遇见throw后，因为异常而导致Fun函数没有执行到delete[]，所以就会造成内存泄漏问题，这样的话对于一个服务器程序来说是不能允许的。

这个只是一个简单的例子，其实还有很多，比如我们在写代码的时候往往会打开一个文件来进行读写操作，然后又是因为异常，导致我们打开的文件没有及时的关闭，从而造成文件描述符泄漏也可以说是内存泄漏。

为了防止这种场景的发生，采用智能指针，通**过对象的构造函数和析构函数来控制对象的持有的堆内存的开辟和销毁**。

## 3.2 智能指针分类

cpp中自带的指针的不断发展，就像上面的提到的那几种，接下来通过代码复原各智能指针的实现，并解析则这种实现带来的问题。

### 3.2.1 auto_ptr实现及问题
```cpp

#ifndef CPP_AUTOPTR_H
#define CPP_AUTOPTR_H

namespace jeason {


    template<typename T>
    class AutoPtr {
    public:
        AutoPtr(T *ptr) : _ptr(ptr) {
        }

        AutoPtr(AutoPtr<T> &autoPtr) : _ptr(autoPtr._ptr) {
            //由于这一步，被拷贝autoptr对象持有的指针已失效
            //因此，该类型的智能指针不是用于原对象和新对象都要用指针的场景，
            //其实有点的独占这个指针的意思
            autoPtr._ptr = nullptr;
        }

        //赋值符号，无非就分为是否改变原值两种情况，
        // 不改变=的入参，就显式声明入参是const
        // 改变=的入参，就显式声明入参是右值，用了右值就要做好右值内部的资源要被转移走的准备
        //从autoptr的设计意图看，资源其实从autoptr转移到了this中
        AutoPtr<T> &operator=(AutoPtr<T> &autoPtr) {

            //提高性能，只在非本身时进行的下面的操作
            if (this != &autoPtr) {
                //下面的操作符合赋值的语义：新值覆盖旧值
                //也就是分两步：
                // 1、清除旧的不用的资源，
                // 2、指向新资源，同时，由于autoptr隐藏的指针独占性
                delete this->_ptr;
                this->_ptr = autoPtr._ptr;
                autoPtr._ptr = nullptr;
            }


            return *this;
        }

        ~AutoPtr() {
            delete this->_ptr;
        }

        T &operator*() {
            return *this->_ptr;
        }

        T *operator->() {
            return this->_ptr;
        }

    private:
        T *_ptr;

    };

}
#endif //CPP_AUTOPTR_H
```

这个智能指针的问题正如代码注释中指出的那样，原来的持有的指针就被置为nullptr，太死板。

于是就出现了boost中scoped_ptr和cpp11中的unique_ptr。

### 3.2.2 scoped_ptr实现及问题
出现这种智能指针，和上面很相似，但是这个处理上面问题的方式很是暴力，直接把赋值与拷贝写成私有声明。就跟本不能用。这个一定程度上减少代码的出错率，但是同时也产生了一定的局限性。

```cpp

#ifndef CPP_SCOPEDPTR_H
#define CPP_SCOPEDPTR_H

namespace jeason {
    template<typename T>
    class ScopedPtr {
    public:
        ScopedPtr(T *ptr) : _ptr(ptr) // 构造
        {}

        ~ScopedPtr() //析构
        {
            if (_ptr != nullptr) {
                delete _ptr;
            }
        }

        T &operator*() {
            return *_ptr;
        }

        T *operator->() {
            return _ptr;
        }

        ScopedPtr(const ScopedPtr<T> &s) = delete; // 防止拷贝
        ScopedPtr<T> &operator=(const ScopedPtr<T> &s) = delete; // 赋值

    private:
        T *_ptr;
    };

}


#endif //CPP_SCOPEDPTR_H
```

### 3.2.3 share_ptr实现及其问题
上面的几种智能指针，几乎都是，一个资源指针只能被一个智能指针对象持有，不利于资源的共享，因此，出现了share_ptr，模仿其使用的引用计数法写一个，STL里的实现才是对的，我写的还是有不少问题的。

```cpp
//
// Created by chenr on 2020-09-01.
//

#ifndef CPPNEWSTARTER_SHAREPTR_HPP
#define CPPNEWSTARTER_SHAREPTR_HPP

template<typename T>
class SharePtr {
private:
    T *_ptr;
    int *_pCount;

    void release() {
        if (1 == *_pCount) {
            delete _ptr;
            delete _pCount;
        } else {
            --*_pCount;
        }
    }

public:
    //clang规范规定，单入参的构造函数都必须用explicit以避免隐式转换
    explicit SharePtr(T *ptr) : _ptr(ptr) {
    }

    //拷贝构造函数，语义上就有不改变input
    SharePtr(const SharePtr<T> &input) :
            _ptr(input._ptr), _pCount(input._pCount) {
        ++*_pCount;
    }

    ~SharePtr() {
        release();
    }


    //赋值符号，从语义看，就是不改变input的
    //并且用新值代替旧只，也就是，先对处理旧的，再接纳新的，
    //也就是，减少旧对象的引用计数，增加新对象的引用计数
    SharePtr<T> &operator=(const SharePtr<T> &input) {
        if (this != &input && this->_ptr != input._ptr) {
            //减少旧对象的引用计数
            --*_pCount;
            if (0 == *_pCount) {
                delete _ptr;
                delete _pCount;
            }
            _ptr = input._ptr;
            _pCount = input._pCount;
            ++*_pCount;
        }

        return *this;
    }

    T &operator*() {
        return *_ptr;
    }

    T *operator->() {
        return _ptr;
    }
};


#endif //CPPNEWSTARTER_SHAREPTR_HPP

```

采用了引用计数的方式，和java中一开始垃圾回收使用的相同，但是引用计数无法解决循环依赖问题，并且我的代码实现，pCount这个变量有是多个对象共享的，且有写入操作，存在明显的多线程问题。