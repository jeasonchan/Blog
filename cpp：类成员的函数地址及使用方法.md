# 1 背景

参考文档：

c++ 类成员函数地址          https://blog.csdn.net/ml232528/article/details/79876915


# 2 实践验证
```cpp
class MyFuntion {
public:

    int echo() {
        return 6666;
    }
};

int main(int argCount, char *args[]) {
    {
        int a = 123;

        //自己显示的加*，表示是地址类型，因为这种下不会发生函数名自动转函数指针的行为
        decltype(leetcode49::hashInt) *aaaa = &leetcode49::hashInt;

        //直接用&取地址
        decltype(&leetcode49::hashInt) bbbb = &leetcode49::hashInt;

        using hashInt_type = std::size_t(*)(int &);
        hashInt_type cccc = &leetcode49::hashInt;

        std::cout << (*aaaa)(a) << std::endl;//打印123
        std::cout << (*bbbb)(a) << std::endl;//打印123
        std::cout << (*cccc)(a) << std::endl;//打印123

        auto fun_address = &MyFuntion::echo;
        MyFuntion myFuntion;
        std::cout << ((&myFuntion)->*fun_address)() << std::endl;
        std::cout << (myFuntion.*fun_address)() << std::endl;

        //这种方式调用成员函数编译都不通过
        std::cout << (*fun_address)(&myFuntion) << std::endl;


        exit(0);
    }

}

```

疑问：

```cpp
std::cout << (*fun_address)(&myFuntion) << std::endl;
```
这种方式编译都不过，那排序、过滤算法是如何直接类成员函数的呢？第一个入参传对象的地址？
