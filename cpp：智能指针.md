# 1 背景
参考文章：


各包装类的接口简单实践    http://www.neohope.org/2018/08/15/%E6%B5%85%E8%B0%88cpp%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88/


浅显介绍     https://www.cnblogs.com/heimazaifei/p/12133715.html


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

通过以上实践发现，CPP的智能指针，与JVM内存中的四种引用方式，强引用、软引用、弱引用，虚引用，有很多相似的地方
