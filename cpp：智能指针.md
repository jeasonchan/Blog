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
        Test(int age, string &name) :
                age(age), name(name) {
            cout << "有参数构造" << endl;
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

        auto_ptr<Test> autoPtrTest(new Test(1, name));

        if (autoPtrTest.get()) {
            autoPtrTest->printDetail();

            autoPtrTest->name = "new name";
            autoPtrTest->printDetail();

            (*autoPtrTest).name = "new name 2";
            autoPtrTest->printDetail();
        }


    }

    void test_share_ptr() {


    }

    void test_unique_ptr() {


    }

    void test_weak_ptr() {


    }


}


int main() {

    jeason::test_auto_ptr();
    jeason::test_share_ptr();
    jeason::test_unique_ptr();
    jeason::test_weak_ptr();


    return 0;
}
```
