# 1 背景

参考文档：

c++ 类成员函数地址          https://blog.csdn.net/ml232528/article/details/79876915

c++类成员函数作为回调函数     https://blog.csdn.net/tieshuxianrezhang/article/details/80635508

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
//编译过不了
std::cout << (*fun_address)(&myFuntion) << std::endl;

//但是，下面通过bind能转为一个可调用的函数对象，注意第一个入参是this
Fun fun=std::bind(&Test::callback,this,std::placeholders::_1,std::placeholders::_2); 

auto new_fun_object = std::bind(&MyFuntion::echo, &myFuntion);
//不知道为啥，Clang/Clion更推荐Lambda写法
auto new_fun_object2 = [ObjectPtr = &myFuntion] { ObjectPtr->echo(); };
```

《深入探索C++对象模型》中提到成员函数时，当成员函数不是静态的，也不是虚函数时，那么我们有以下结论： 

（1）&类名::函数名 获取的是成员函数的实际地址；

（2）对于类成员函数fun来讲，obj.fun()编译器转化后表现为fun(&obj)，&obj作为this指针传入； 

（3）无法通过强制类型转换在类成员函数指针与其外形几乎一样的普通函数指针之间进行有效的转换。这条解释了为啥上面的示例编译过不了

要看一下bind的源码，看看是怎么把一个成员函数转为一个脱离了对象的函数对象的。
