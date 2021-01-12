# 1 背景
使用基于哈希值的容器时，如果自定义的类有计算哈希值的需求，那自己自然必须实哈希值计算和对象相等的代码，在Java中，直接有equals和hash两个类成员方法；CPP哈希容器，通过在哈希容器构造时指定计算哈希值和等于判断的逻辑的函数对象或者函数指针，并没有将这两个函数当作对象的成员方法，不是那么的面向对象，但也符合CPP的设计哲学，不需要为自己不用的东西损失性能。


参考文档：

C++自定义哈希函数以及烦人的const   https://www.jianshu.com/p/835e2c5c3a58



# 2 

# 3 代码实践


```cpp
#include<vector>
#include<string>
#include<unordered_set>
#include<unordered_map>
#include<algorithm>


//===========================================================================================================================================================
//===========================================================================================================================================================
//===========================================================================================================================================================
//===========================================================================================================================================================


template<>
struct std::hash<std::unordered_set<int>>{
    using type=std::unordered_set<int>;

    size_t operator()(const type input) const noexcept{
        return static_cast<size_t>(1);
    }
};

template<>
struct std::equal_to<std::unordered_set<int>>{
    using type =std::unordered_set<int>;
    bool operator()(const type &o1,const type & o2)const noexcept{
        return o1.size()==o2.size();
    }

};


size_t myHash(const std::unordered_set<int> input) noexcept{
    return static_cast<size_t>(1);
}

bool myEqual(const std::unordered_set<int> &o1,const std::unordered_set<int>  & o2) noexcept{
    return o1.size()==o2.size();
}

using myType=std::unordered_set<int>;

std::unordered_set<myType> record;
std::unordered_set<myType,std::hash<myType>> record01();


size_t hash_vec(const std::vector<int> input) noexcept{
    return static_cast<size_t>(1);
}




//以下两种模板的传参，没有利用自动推导，只要模板那里和后面用到的的类型对应起来就能编译通过、运行
int robotSim(std::vector<int>& commands, std::vector<std::vector<int>>& obstacles) {
    std::unordered_set<std::vector<int>, decltype(hash_vec) *> S(obstacles.begin(), obstacles.end(), 10000, &hash_vec);
     /* ... process ... */
    return -1;
}

int robotSim2(std::vector<int>& commands, std::vector<std::vector<int>>& obstacles) {
    std::unordered_set<std::vector<int>, decltype(&hash_vec)> S(obstacles.begin(), obstacles.end(), 10000, &hash_vec);
     /* ... process ... */
    return -1;
}


void fake_main(){
    std::vector<int> input{};

    auto fun=&hash_vec;

    //这是对函数地址先解引用，再进行函数调用
    auto result1=(*fun)(input);

    //直接把函数地址当作函数名时，自动进行
    auto result2=fun(input);
}


//===========================================================================================================================================================
//===========================================================================================================================================================
//===========================================================================================================================================================


```
