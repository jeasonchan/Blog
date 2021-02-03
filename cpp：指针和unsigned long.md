# 00 背景

参考文档：

为什么Linux内核常常用unsigned long来代替指针  https://mp.weixin.qq.com/s/28I6GZr0OKfXcQ8SIu9cdg


# 01 代码实践
先说结论：
内核中，很多地方都只关心指针本身的值，根本不需要解引用，为了防止有人尝试解引用，在内核中完全不需要对指针解引用的地方，都直接用unsigned long来表示指针的值，CPP里得通过C语言的风格 或者 reinterpret_cast 进行强制转换。


```cpp
#include<iostream>



/**
 * @brief main
 *
 * 研究一下进程中指针的值
 *
 * @param argc
 * @param argv
 * @return
 */
int main(int argc,char* argv[]){
    using namespace std;

    int a=123;
    int* p=&a;

    cout<<p<<endl;
    cout<<(unsigned long long)(p)<<endl;
    cout<<reinterpret_cast<unsigned long long>(p)<<endl;
    //cout<<static_cast<unsigned long long>(p)<<endl;  编译器不允许
    cout<<*p<<endl;
    /*
     输出：
     0x61fe14
     6422036
     6422036
     123
    */

    return 0;

}
```
