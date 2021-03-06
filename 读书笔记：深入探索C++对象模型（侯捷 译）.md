# 0 背景
读书的过程中，除了阅读原始书籍之外，同时还参考了部分老哥的笔记。


参考文章：

xiezhw3/Inside-the-C-plus-plus-Object-Model-note   https://github.com/xiezhw3/Inside-the-C-plus-plus-Object-Model-note/tree/master/%E7%AC%94%E8%AE%B0


这本书主要讲两个方面的问题：

1. 一个真实的运行时的CPP对象在内存中的样子？
2. 编译器为了维护CPP的面向对象的"生态"，对我们的源码做了什么？

# 1 关于对象


## 1.1 C++对象的内存模型


### 1.1.1 简单对象模型

巴拉巴拉

### 1.1.2 表格驱动对象模型

巴拉巴拉

### 1.1.3 C++对象模型
是C++最初的内存模型：

1. 非静态的数据成员存在于每个对象实例中，即占用对象实例的内存
2. 静态的数据成员对象不会占用对象实例的内存
3. 静态和非静态的的成员函数不占用对象实例的内存
4. 虚函数通过以下机制来实现：有虚函数的类都有自己的虚函数表，子类覆写了父类虚函数后，虚函数表中指向的函数就变成了子类的函数；每个有虚函数的对象实例的第一个成员对象都是指向那个虚函数表的指针，叫虚表指针，虚表指针有且只有一个。

这个对象模型的优点：

1. 能抽出来共用的部分基本都共用了，已经有很高的空间和存取效率了

缺点：

1. 某类增加了非静态数据成员后，编译生成的内存模型必然发生了变化，这就导致使用该类的程序都要重新编译，也就是没有做到 二进制兼容性；表格驱动的对象模型就没有这个问题，对象里面只有两个指针的空间占用。

### 1.1.4 继承关系在对象模型中的体现
C++支持单一继承、多重继承、虚继承（http://c.biancheng.net/view/2280.html，解决菱形继承的场景中公用基类的冗余问题），那派生类的这种派生关系的在内存模型中是如何体现的？


对于简单对象模型，用一个slot指向父类即可，优点是父类结构发生变化，对子类无影响，能保证子类的二进制的兼容性。缺点是，用指针作为一个中间层，要用指针跳转一次，影响性能。


对于表格驱动的对象模型，模仿虚函数的实现过程造一个基类表，也就是一个实例对象会有三个指针了：指向数据成员表的指针、指向函数表的指针、基指向类表的指针。


简单对象模型和表格驱动对象模型在实现继承时，都无法避免指针间接引用带来的性能损耗。C++**最初的**对象模型为了实现较高的性能，直接将父类的成员放到子类中，也就是对象实例的内存分布上，**越靠前部分的内存，越是父类的非静态数据成员。**缺点，自然是的父类的改变会导致子类的二进制兼容问题。

C++对象模型为了实现虚继承，和虚函数的处理方式类似，也使用了间接的方式。最原始的处理方法是，使用一个虚继承表。

### 1.1.5 对象模型如何影响程序
编译器会偷偷修改的我们写的代码，比如RVO优化等等，内存模型对我们写的代码同样有很大的影响，比如：虚函数的调用

```cpp
class X {
    virtual void handle(){
        //do sth
    }

};



void a_function(){
    X x1;
    X p_x*=new X;

    //编译器转译为：
    //handle(&x1)
    x1.handle();
    
    //编译器转移为：
    //(*(p_x->vptr[2]))(p_x)
    //即，从vptr指向的虚函数表中，找到第二个函数的地址，再对函数地址解引用，实现函数调用
    p_x->handle();


    //虚函数的多态只能通过指针和引用，直接通过实例名无法实现，所以上面通过对象直接调用需函数时没有走虚函数表的查找过程

}

```

可见，对象内存模型对编译器的影响十分重大


## 1.2 关键词带来的差异
C和C++中的关键词带来的差异……这本书感觉讲得乱七八糟的……是侯捷翻译的不好吗？？？读完这一小结后，发现是在讲struct关键字在两种语言使用过程中的差异。

C++中的struct和class关键字，除了默认的成员可见性不同，其他完全相同，而C中的struct只能含有数据成员对象。

个人觉得pure cpp就不要强行使用C风格的东西了，真的要强行混合使用C的struct和cpp的对象时，单独将C的结构体抽出来，封装成一个纯粹的C的struct。


## 1.3 对象的差异

C++的程序设计模式直接支持多种模式：面向过程、面向对象、泛型编程（STL就是泛型编程的典范）。

要想实现多态，必须通过指针或者引用进行调用，比如：

```
//走虚函数表的
X *p=new X();
X &r=(*p);

p->fun();
(*p).fun();
r.fun();



//不走虚函数表，直接调X::fun(X*)这个函数
//这一语句其实是初始化x
X x=*p; 
x.fun()
```

一个实例对象的内存占用大小包括：
1. 所有非静态成员对象的大小
2. padding所加进去的字节，padding的目的就是将对象所占内存大小调整为最大寻址地址的整数倍。比如，编译为32位的程序，则应该为32位（即4字节）的整数倍
3. 处理各种virtual带来的内存占用，也就是几个固定大小的指针。


### 1.3.1 指针的类型
指向不同对象、基本类型的指针，有什么不同？从内存的角度看，完全一样，都只是持有一个虚拟地址空间的地址而已，指针类型只是告诉编译器从地址往后取多少字节是这个对象的内存地址范围,编译器知道类型后才能进行各种操作。

所以，编译器拿到void\*类型的指针，毫无乱用，只是拿到了一个对象的起始地址，这个对象占了多少自己并不知道，然后就什么也不能做了，除非做一些强制转换再使用。

所以，转换（cast）其实算一种编译器指令，没有改变起始地址，而是告诉编译器读取的字节长度，本质上，cast操作是改变了编译器读取并解释字节的方式。

### 1.3.2 考虑多态



# 2 构造函数相关的语义



## 2.4 初始化列表
何时使用initialization list才有意义，然后解释list内部的真正操作是什么，然后我们再来看看一些微妙的陷阱。

以下情况不得不使用初始列表对成员变量进行初始化：

1. 当初始化一个 referencemember时；
2. 当初始化一个 const member时；
3. 当调用一个 base class的 constructor，而它拥有一组参数时；
4. 当调用一个 member class的 constructor，而它拥有一组参数时。

因为，在构造函数体内对成员变量进行=时，其实是正真的赋值操作，而不是 T t=xxx  这种定义并初始化操作。

假设有一个简单的对象，有且仅有String和int成员对象，如果在无参构造函数的函数体中对这两个成员进行**赋值（没错，是赋值，根本不是初始化）**，则这个无参构造函数会被编译器改成：

![默认构造、拷贝构造综合运用.PNG](./resources/默认构造、拷贝构造综合运用.PNG)

PS：基础数据类型是非常nice的类型，不需要构造函数，直接按位拷贝即可。

**编译器会一一操作initialization list，以成员变量声明的顺序在constructor之内安插初始化操作，并且在任何explicit user code之前。**



# 3

## 3.1


## 3.2


## 3.3 对象成员的存取

```cpp
Point3d origin, *p=&origin;

origin.x=0.0;

p->x=0.0;
```

通过实例名和通过指针读写对象成员x的成本分别是什么？有什么异同？

其实要分多种情况来讨论：

1. x是静态成员对象
2. x是非静态成员duixiang
3. 对象本身是一个完全独立的类
4. 对象存在多重继承和虚继承的情况

接下来就针对这些情况分别探讨对成员对象的读写成本


### 3.3.1 静态成员
每一个static  data  member只有一个实例，存放在程序的data  segment之中。每次程序参阅（取用）static  member时，就会被内部转化为对该唯一extern实例的直接参考操作。

对于静态成员变量的存取，是C++语言中"通过一个指针和通过一个对象来存取  member，结论完全相同”的唯一一种情况。这是因为“经由  member  selection  operators（译注：也就是“.”运算符）对一个static  data  member进行存取操作”只是文法上的一种便宜行事而已。member其实并不在class  object之中，因此存取static  members并不需要通过class  object。

取一个static  data  member的地址，会得到一个指向其数据类型的指针，而不是一个指向其class  member的指针，因为static  member并不内含在一个class  object之中。

那多个类都有同样名字的静态成员怎么办呢？

1. 一个算法，推导出独一无二的名称。 
2. 万一编译系统（或环境工具）必须和使用者交谈，那些独一无二的名称可以轻易被推导回到原来的名称。

### 3.3.2 非静态成员
显然，非静态成员必定是属于一个对象实例的，要想读写必须显式或者隐式的通过对象实例进行。隐式是指，有时候我们的省略this指针，但是编译器生成的代码还是会偷偷的给我们补全this。

欲对一个nonstatic  data  member进行存取操作，编译器需要把class  object的起始地址加上data  member的偏移位置（offset）。

```cpp
origin.y==0.0;


&origin.y;
//经过编译器在编译期的改造，会变成：
//注意！！使用的是&Pointed3d::y，返回的是相对于整个对象开始地址的相对地址
&origin+(&Pointed3d::y-1);
```

为什么这个offset要-1呢？？？？因为&Pointed3d::y 的语义是总是+1的，因此，这真正相当于this的地址应该-1。**注意，这都是直接通过实例对象进行访问的，而不是指针**

每一个nonstatic  data  member的偏移位置（offset）在编译时期即可获知，甚至如果member属于一个base  class  subobject（派生自单一或多重继承串链）也是一样的。因此，存取一个nonstatic  data  member，其效率和存取一个C  struct  member或一个nonderived  class的member是一样的。


对于本小节一开始提出的问题，以下两种有啥区别：

```cpp
Point3d origin, *p=&origin;

origin.x=0.0;

p->x=0.0;
```
**对于x不是虚继承父类的成员对象时，通过实例名和指针访问成员对象，效率没有啥区别，因为和还是可以在编译期就确定offset的大小。**

当x是虚继承父类的成员对象时，我们不能够说pt必然指向哪一种class  type（因此，我们也就不知道编译时期这个member真正的offset位置；就跟虚函数表的处理方式类似，虚标指针在实例初始化时，会在构造函数的前部分中自动指向相应的虚函数表），所以这个存取操作必须延迟至执行期，经由一个额外的间接导引，才能够解决。但如果直接使用origin，就不会有这些问题，其类型无疑是Point3d  class，而即使它继承自virtual  base  class，members 的offset位置也在编译时期就固定了。**所以，该种情况下，用指针时会多一层虚基类指针的访问。**

## 3.4 继承和成员对象
在C++继承模型中，一个字类对象实例所表现出来的东西（也就是在内存中的占用情况），是其自己的members加上其base  class（es） members的总和。有虚继承的情况，都要单独讨论。



### 3.4.1 单一继承不含虚函数（也就是不考虑多态）
一般而言，选择某些函数做成inline函数，是设计class时的一个重要课题。**仅仅是为了减少函数调用的开销吗？？**


单一继承得到的子类实例内存分布符合3.4一开始说的那个规律，但是，由于字节对齐机制的存在，**父类中用于对齐的无用字节仍然会带到子类中，最终导致子类存在无意义的膨胀现象**。因为C++标准规定，子类中出现的父类实例必须保证其完整原样性。如果不做这样的强制规定，CPP面向对象体系中，对于"简单对象"的按位拷贝的这种十分朴素的机制就会被破坏。


### 3.4.2 单一继承包含虚函数（也就是考虑多态）
先来总结一下，如果简单的为一个类增加一个虚函数，编译器会多做哪些额外的工作：
1. 生成虚表，用来存放实现了虚函数的地址，每个实现类一个表，在编译器就已经生成这个表了；同时，这个虚表不仅仅包含虚函数的地址，还包含几个曹用来支持RTTI
2. 编译器偷偷为有虚函数的类增加一个指针，指向当前类的虚表；目前的编译器的主流实现是，放在对象最前面；很早以前放在最后面，可以和C结构体兼容，但是目前的抽象类等特性太多，放在最前面更有利于编译器实现OO特性
3. 编译器对构造函数做修改，保证在对象构造时首先完成虚表指针的初始化，即初始化为虚表的地址
4. 编译器对构造函数做修改，取消虚表指针指向虚表

这些相对于原来的类来说，多出来的动作都是性能消耗，比较适用于多态十分多的情况，如果子类仅仅是一个类则收益可能不大。


### 3.4.3 多重继承
在单一继承中， 看到base class 和 derived class 的 objects 都是从相同的地址开始， 其间差异只在于derived object 比较大， 用以多容纳它自己的 nonstatic data members。但是，某一个子类突然增加了一个虚函数，而编译器必然会在编译期给子类偷偷增加一个vptr成员变量（而且现在的编译器都是直接加在最前面的），子类多出来这样一个成员变量就影响了用户定义的成员变量的offset，这种情况如何解决的呢？

把一个含有虚函数derived object 转换为其base类型， 就必然需要编译器的介入， 用以调整地址（ 因vptr插入之故）。在既是多重继承又是虚拟继承的情况下， 编译器的介入更有必要。

多重继承的复杂度在于 子类 和 其父类 乃至 和 其父类的父类 之间的复杂关系。

假设有如下继承关系：

```cpp
class Base1{

}

class Child1:public Base1{

}


class Base2{

}


//重点来了，开始多重继承
//首先根据多重继承的声明顺序
//Child2的实例对象，在内存上的分布依次是：Child1的数据、Base2的数据
//其中，Child1的数据的开头又是Base1的数据
class Child2:public Child1, public Base2{

}
```

可见，对于多重继承中，由于内存分布和多重继承的声明的顺序有重大的关系，子类向多重继承中的第一父类转型时，很简单，因为起始地址是一样的，只是占用的长度不同，和单一继承一毛一样；

对于向多重继承中的第二甚至第三父类转型时，就比较复杂了，起始地址就必须由编译器做一些手脚了，具体做什么手脚呢？

```cpp
Child2 child;

Child1 *p_Child1;
Base1 *p_Base1;
Base2 *p_Base2;

//当进行以下的转型时，编译器会偷偷及进行相应的操作：

//这两个的编译器不需要做额外的事情，本质上就是赋值为this指针指向的地址
p_Child1=&child;
p_Base1=&child;

//显然，这个语句的本意是希望只使用属于Base2的那部分数据结构
p_Base2=&child;
//编译器会在内部做这样的转化
p_Base2=(Base2*)(  ((char*)&child)+sizeof(Child1)  );
```


以上编译器在内部所作的优化其实是因编译器对多重的实现来决定了，主流的编译器在实现多重继承时，基本是直接根据多重继承的声明顺序在内存上排布各个父类的成员变量。对于虚继承这种需要安插虚继承指针的情况，下一小结进行说明。

### 3.4.4 虚继承
虚继承需要编译器做的东西太多了，各个编译器对虚继承的实现也都有些不同，但大体思路是相同的：
1. 对于与非虚继承的父类成员对象，和普通的继承一样，有固定的offset
2. 对于虚继承的部分，由于其位置在每次派生时都会发生变化（比如，有固定offset的成员加入），所以，虚继承的部分只能间接读写，间接读写的方式因编译器的实现而不同

（具体的部分书上也讲的比较少，略过）


## 3.5 继承、封装、多态对类成员的操作效率对比
本小节，作者通过对：
1. 简单的float变量
2. C结构体中的float变量
3. C++类中的float变量
4. C++类中通过inline函数对float变量
5. 单一继承情况下，C++类中通过inline函数对float变量
6. 多重继承情况下，C++类中通过inline函数对float变量
7. 虚继承情况下，C++类中通过inline函数对float变量

场景，循环执行1000万次操作，计算时间消耗，最终发现，除了虚继承的情况，开启了编译器优化的程序，执行时间都没有差别。可见，虚继承带来的间接性，压抑了"把所有运算都移往寄存器执行"的优化行为。

## 3.6 指向成员变量的指针
顾名思义，就是类中成员对象的地址，知道这个地址有两个作用：
1. 知道编译器是把vptr放在最前面还是最后面
2. 知道编译器是不是直接按照class中access section声明的顺序排布内存

比如，有下面的这个类声明：

```cpp

class Point3d{
public:
    //再次强调下，虚析构函数函数的重要性，防止多态场景中不释放子类的资源
    virtual ~Point3d();

protected:
    static Point3d origin;
    float x,y,x;

}
```

显而易见，在Point3d的实例内存分布上，x y z三个成员变量是连着的，静态的成员不在实例中，唯一一因编译器不同而不同的就是vptr的位置，所有的编译器的实现，不是放在开头就是放在末尾。

直接用例子说明几个重要的问题：
1. &Point3d::x的语义  (和具体对象无关吗？？？**要代码实践一下**)
2. &obj.x的语义


```cpp
#include<iostream>
#include<string>

/*
验证 指向成员对象和成员函数的指针

直接根据

*/

namespace  jeasonchan{
using namespace std;

class Person
{
public:
    Person():age(666),name("default") {}

    int age;
    string name;

    void saySth_copy(){
        cout<<"name:"<<name<<endl
           <<"age:"<<age<<endl;
    }

    void saySth(){
        cout<<"name:"<<name<<endl
           <<"age:"<<age<<endl;
    }

};

}


int main(int argc,char* argv[]){
    using namespace jeasonchan;
    Person p;

    std::cout<<&(p.age)<<std::endl;  //输出age这个int 在进程虚拟地址空间中的地址

    int Person::*offset_of_Person_int=&Person::age;//age从类实例起点开始计算的偏移量，单位是字节数
    std::cout<<offset_of_Person_int<<std::endl;//打印this开始的偏移量
    std::cout<<p.*offset_of_Person_int<<std::endl;//根据偏移量，从类的起始点开始取值


    void (Person::*offset_of_Person_saySth)()=&Person::saySth;
     std::cout<<offset_of_Person_saySth<<std::endl;//成员函数不占用任何对象实例的内存呢，输出1
    (p.*offset_of_Person_saySth)();


    //利用thread时，除了可以传递lambda外，还可以：
    std::thread thread01(&Class::funName,arg...);

    return 0;
}


```
指向成员对象和成员函数的指针   https://blog.csdn.net/alex1997222/article/details/81983073



# 4 函数的语义
本章讲编译器对函数本身和函数调用 偷偷做的事情，比如mangling、成员函数、非成员函数、静态函数等。


## 4.3 函数调用的效能对比及分析
对比范围：
1. nonmember friend function
2. member function 
3. virtual member function 在单一、虚拟、多重继承三种情况

根据编译器对各种函数的处理方式，nonmember、static member或nonstatic member函数都被转化为完全相同的形式，所以效率可以认为是完全一致的。即为：

1. nonmember function和static member function的区别在于static function是归属于类的方法，是一个不依赖于对象实例的方法（也就是入参没有隐藏的this指针），而前者则不从属于任何一个类别。从运行效率上来说都可以从代码段中直接访问函数，过程相同，别无二致。

2. non static member function是会被编译器自动插入this指针，做mangling签名修饰甚至是返回值优化，其最后就摇身一变成为了nonmember function因此其在性能上也是等同于上述两类函数的。


inline函数不只能够节省一般函数调用所带来的额外负担，也提供了程序优化的额外机会，比如，编译器将“被视为不变的表达式（expressions）”提到了循环之外，只计算一次。


另外有一个奇怪的现象：每多一层继承，virtual function的执行时间就有明显的增加。其实并不是虚函数调用的开销随着继承加深而增大，而是测试过程中，用的对象的构造函数，随着继承加深而变得更加复杂：

1. 复杂性，体现在，实例对象必须先构造父类实例，父类实例里有vpr，所以，每多一层继承，就会多增加一个额外的vptr设定。

2. if(this||this=new (sizeof(*this))){ //一些初始化代码 }  每个类的构造函数，编译器都会自动把我们的代码放入这个判断的执行体中，随着继承加深，构造对象时判断执行的次数变得很多

## 4.4 指向成员函数的指针

```cpp
void (Person::*offset_of_Person_saySth)()=&Person::saySth;
std::cout<<offset_of_Person_saySth<<std::endl;//成员函数不占用任何对象实例的内存呢，输出1
(p.*offset_of_Person_saySth)();

```
指向成员函数的指针很容易，但是，对于指向成员的虚函数 的指针就比较复杂了。

### 4.3.1 编译器对的成员虚函数指针的处理
先说结论：通过指向成员虚函数的指针（其实是offset），再通过子类取调用，同样能实现多态。代码示例：

```cpp
class Base{
public:
    virtual void do(){cout<<"Base";}
}


class Child public Base{
public:
    void do(){cout<<"Child";}
}


#尝试使用指向成员函数的指针

void(Base::*offset)()=&Base::do;
Base *ptr=new Child;

ptr->do();//打印 Child
(ptr->*offset)();//还是打印 Child
```

可见，尽管offset一开始取得是&Base::do，但是，通过子类指针调用同样能实现多态。

先个人**猜测**一把：一开始的&Base::do，记住了vptr在类实例中的相对地址（主流编译器都是放在第一个）和索引值，当Child重写虚函数时，子类实例中的vptr指向Child自己的虚表，再根据之前记住的索引值就可以找到被重写的函数了。

果然！！！！！对于成员虚函数的取地址操作，就是直接取的索引值，然后真正调用时会这么用：
```cpp

//已经考虑到编译器自动添加了this指针这个入参
(*ptr->vptr[(int)offset])(ptr);

```

### 4.3.2 多重继承下指向成员函数的指针

（猜测，成员函数最终都会被编译器抓换成普通函数，再加上虚表指针这些，顶多和多重继承的成员对象存取一下，this+offset 得到 第二、第三、第N父类的起始地址）

## 4.5 内联函数
```cpp
class Example{
private:
    int age;
public:
    inline void age(int & _age){this->age=_age;}
    inline int age(){return this->age;}
}

```
把这些存取函数声明为inline，我们就可以继续保持直接存取members的那种高效率——虽然我们亦兼顾了函数的封装性。

inline只是对编译器的建议，某些编译器会根据这些指标打分，最终确定需不需要内联。cfront有一套复杂的测试法，通常是用来计算assignments、function calls、virtual functioncalls等操作的次数。每个表达式（expression）种类有一个权值，而inline函数的复杂度就以这些操作的总和来决定。

编译器处理inline函数一般有两个阶段：
1. 根据函数定义和掉哦那个分析是否可以内联。

被判断不可成为inline，它会被转为一个static函数，并在“被编译模块”内产生对应的函数定义

2. 真正的 inline函数扩展操作是在调用的那一点上。这会带来参数的求值操作（evaluation）以及临时性对象的管理

大部分编译器厂商（UNIX和PC都有）似乎认为不值得在inline支持技术上做详细的讨论

### 4.5.1 形式参数
每一个形式参数都会被对应的实际参数取代，但是，替代的方式可能不同，尤其是当入参直接是常量时，举个例子：

```cpp
inline int min(int i,int j){
    return i<j?i:j;
}

//下面尝试对这个内联函数进行调用
int val_1=1;
int val_2=2;
int result;

result=min（1，2）；
//直接转化，在编译期间就能确定初始值
result=1;


result=min(val_1,val_2);
//转化为
result=val_1<val_2?val_1:val_2；


result=min(fun(),fun2());
//转化为
int temp1=fun();
int temp2=fun2()
result=temp1<temp2?temp1:temp2

```

### 4.3.2 局部变量
一般而言，inline函数中的每一个局部变量都必须被放在函数调用的一个封闭区段中，拥有一个独一无二的名称



# 5 构造、析构、拷贝的语义
是否提倡在抽象基类中声明成员变量？
1. 不提倡！接口定义要和实现分离
2. 提倡！把公共的被共享的数据抽取放入基类中没啥问题


## 5.1 无继承情况下的对象构造
（暂时不看……）



# 6 运行期语义
用隐式转换的例子引入运行期中所做的事情。

## 6.1 对象的构造和析构
离开作用域时，对象的析构其实是靠编译器吭哧吭哧的"在每个出口点显式调用对象的析构函数实现的"。

C++中，有这样一条规则：当用户试图在栈上创建对象时，编译器会查找匹配且可以访问的构造函数和析构函数，如果其中一个无法访问，编译就会报错。

所以，为了不让类在栈中创建，我们可以将析构函数声明为private。同时，为了保证delete堆对象"正常工作"，自己需要写一个destroy函数，间接调私有的析构函数。


为了不让类在堆中创建，我们可以将new和delete操作符声明为私有即可。

会把object尽可能放置在使用它的那个程序区段附近，这么做可以节省非必要的对象产生操作和摧毁操。

### 6.1.1 全局对象
在还没有执行到main函数时，全局对象的内存空间已经在进程的数据段申请好了，并且bit位全是0，但constructor一直要到程序启动（startup）时才会实施，析构在main函数退出前进行析构。

对全局变量的初始化和析构策略：
1. 为每一个需要静态初始化的文件产生一个_sti（）函数，内含必要的constructor调用操作或 inline expansions。
2. 类似情况，在每一个需要静态的内存释放操作（staticdeallocation）的文件中，产生一个__std（）函数。
3. 提供一组 runtime library“munch”函数：一个_main（）函数（用以调用可执行文件中的所有__sti（）函数），以及一个 exit（）函数（以类似方式调用所有的__std（）函数）。

最终来看，也就是在main()函数的最前面和最后面加入了对全部变量的构造和析构。



支持“nonclass objects的静态初始化”，在某种程度上，是支持virtual base classes 的一个副产品。因为：
1. 指向虚基类的vbcPoint是在运行期确定的，无法在编译期完成
2. vbcPoint是个指针，是个no-class object
3. 全局变量要是有虚基类的指针，就不得不在运行期对该虚基类的指针进行初始化
4. 因此，no-class 对象的静态初始化，"拓展"到了运行期


使用被静态初始化的objects，有一些缺点。例如，如果exception handling 被支持，那些objects将不能够被放置于try区段之内，但是，在try catch外先定义，try catch中再赋值 或者 也没啥不可以。

### 6.1.2 局部静态对象
在函数中 或者 文件中的 static对象，编译器能保证如下语义：
1. 仅在第一次使用时构造这个local static object
2. 按照程序中所有static对象构造出来的顺序，逆序的析构local static object

### 6.1.3 对象数组
```cpp
//C风格的数组
Objetc arrayObject[10];

```
如果一个class既没有定义一个 constructor也没有定义一个destructor，我们只要配置足够内存以存储10个连续的Objetc元素即可。


实际上，声明/定义一个的具名对象数组（栈中）和匿名对象数组（堆中），都会调用的编译器的vec_new或者vec_vnew函数，入参是对象的无参构造函数的地址。

对于仅有有参构造函数，且入参是带默认值的类，为了因应对vec_new只能调用无入参的函数的情况，编译器会偷偷生成一个真正的无参构造，只是简单的调用带带默认值的有参构造。


## 6.2 new和delte运算符
new运算符其实是分为两步执行的：
```cpp
//原始
int *p=new int(5);

//编译器将new运算符转换的伪码，真是过程分为两步：
//1、通过适当的new运算符函数实例，配置所需内存
//和malloc一样，只申请空间，无额外操作，内存的bit位为不确定的状态
int *p=__new(sizeof(int));

//2、在分配的内存空间上设定初始化
*p=5;

//连贯一起来应该这么写
int *p=nullptr;

if(p=__new(sizeof(int))){
    //内存申请成功才进行初始化，不过，默认的new申请不成功过应该是抛异常吧……
    int *p=nullptr;
}

```

delete操作也是类似的，对于类 实例，转化如下：
```cpp
Object *p=new Oject;
delete p;

//
if(p!=nullptr){
    //其实应该是，Object::~Object(p)，因为成员函数最终会通过mangling的方式成为一个普通函数，并被偷偷添加一个this指针
    //释放本身持有的资源，一般就是指针指向的堆内存
    p.~Object();

    //释放类实例本身占用的内存空间
    __delete(p);
}

```
也就是，delete运算符会先调用类的析构函数，再将占用的堆内存为可用状态。

### 6.2.1 针对数组的new语义
对于基础数据类型和没有无参构造函数的对象/结构体，new T[const N]这样的数组时，其实只会发生  new (N*sizeif(T))  这样的过程，并不发生通过构造函数进行初始化的过程，为什么呢？因为类型T很简单，是bitwise的。

对于有无参构造的类型T，无论是程序员自己显式声明出来，还是编译器根据我们的类声明（比如，类成员有string这种本身就有默认构造函数的，memberwise的类，编译器会必定偷偷在构造函数里对memberwise成员做初始化）自动生成的，在new数组时，vec_new才有**可能**的被调用。

问题来了，deletep[] point  时，delete是怎么知道这个数组有多少元素？？？

vec_new（）所传回的每一个内存区块配置一个额外的word，然后把元素个数包藏在那个word之中。通常这种被包藏的数值称为所谓的cookie（小甜饼）。所以，一个字节的能表示的大小就是new出来的数组的最大元素个数？


注意！！！！！！！！！！！！！！！

数组的地址，其实就是首元素的地址，如果把首元素的地址拿去向上转型成父类，再拿这个父类型的指针去delete[\]，会出现问题！因为，delete[\]的会根据这个类型和数组的大小去释放内存，这就最终导致数组释放不完整，且 元素的析构并不能正确持有的堆内存。

但是，对于普通的对象指针，去delete转型得到的父类指则针完全没有问题，因为，析构函数是virtual的情况下，通过指针或者引用去析构，会直接走虚表指针调用真正的析构函数，而不是简单的根据指针类型。

### 6.1.4 放置new的语义
经常被拿来实现内存池，提高内存的复用率，减少内存碎片。


有个要点，重新在内存池上构造新对象前，必须使用指针调用析构函数的方法，而不能直接用delete，因为，delete会把内存归还给进程的虚拟空间，等于破坏了内存池。

## 6.3 临时对象
右值


# 7 CPP对象模型总览
 
## 7.1 模板
所有与类型有关的检验，如果牵涉到template参数，都必须延迟到真正的实例化操作（instantiation）发生，也就是，实例化时才会检查范型绑定的类型究竟有没有相应成员函数/成员数据。


类模板中，如果用到的函数和泛型无关（只需要考虑函数名和入参，且不需要考虑返回值），则类模板中所使用的函数可直接决议出来；这叫 scope of the template declaration。

如果用到的函数和泛型有关，则需要在类模板特化时，才结合泛型类型进行决议。这叫 scope of the template instantiation。

### 7.1.1 成员函数的实例化
成员函数的实例化，其实就是**从函数模板，变成模板函数**，这是很复杂的问题，编译器为了支持这个特性，主要解决三个问题：

1. 编译器如何找到函数的定义

2. 编译器如何只实例化用到的函数

3. 编译器如何避免函数模板在多个.o文件中都被实例化


C++支持template的原始意图是建立一个由使用者导引的自动实例化机制（use-directed automatic ins tantiation mechanism），既不需要使用者的介入，也不需要同一目标的模板函数有多次的实例化行为。（然而，很多时候我们并不需要的匹配任何模板或者指向匹配特定的模板进行实例化，就需要用到比如SFINE这样的小技巧。）。

（鸽了，有点复杂……因为自己确实不经常使用模板……等稍微踩坑了再来补习）


## 7.2 异常处理

为了实现异常处理，编译器必然需要对程序员写的try catch结构做出一些反应，包括：

1. 找出catch子句，以处理被抛出来的 exception，追踪程序堆栈中的每一个函数的目前作用区域
2. 编译器必须提供某种查询exception objects 的方法，以知道其实际类型（这直接导致某种形式的执行期类型识别，也就是 RTTI）
3. 还需要某种机制用以管理被抛出的 object，包括它的产生、存储、可能的析构（如果有相关的destructor）、清理（clean up）以及一般存取。
4. 可能有一个以上的objects同时起作用

然而，和Java不同，CPP认为有些错误并不是异常，而是业务逻辑错误，比如 除以0 ，并不抛出任何异常，而是直接terminate()，这意思是要让程序员手动实现逻辑判断这种情况。

当一个exception 被抛出去时，控制权会从函数调用中被释放出来，并寻找一个吻合的catch子句。如果都没有吻合者，那么默认的处理例程terminate（）会被调用。当控制权被放弃后，堆栈中的每一个函数调用也就被推离（popped up）。这个程序称为unwinding the stack。在每一个函数被推离堆栈之前，函数的local class objects的destructor会被调用。


决定 throw是否发生在一个 try区段中，一个函数可以被想象为好几个区域，编译器必须标示出以上各区域，并使它们对执行期的exceptionhandling 系统有所作用。

当throw操作发生时，目前的program counter值被拿来与对应的“范围表格”进行比对，以决定目前作用中的区域是否在一个try区段中。

每一个被抛出来的 exception，编译器必须产生一个类型描述器，对exception的类型进行编码。类型描述器（type descriptor）是必要的，因为真正的exception是在执行期被处理的，其object必须有自己的类型信息。RTTI正是因为支持EH而获得的副产品。

执行期的 exception handler会将“被抛出之object的类型描述器”和“每一个cause子句的**类型描述器**”进行比较，直到找到吻合的一个，或是直到堆栈已经被“unwound”而 terminate（）已被调用。

每一个函数会产生出一个exception 表格(counter-range表格)，它描述与函数相关的各区域、任何必要的善后处理代码（cleanup code，被localclass object destructors调用）以及catch子句的位置（如果某个区域是在try区段之中的话）

### 7.2.1 实际对象被抛出时发生什么
throw的一个对象时，**这个对象首先被放进异常栈中**，之后再用异常栈中的对象去初始化匹配上的catch的形参，catch的形参时引用类型的，则会真正修改异常栈中的那个对象；如果，catch的形参时值类型的，那必然是触发拷贝构造了。

那放进异常栈的那个exception object是拷贝得来的，还是就是原来对象的引用呢？对于全局变量，是拷贝一份放进异常栈；对于局部变量，好像是放引用。**有待代码实践确认！！！！！！！！！！！！！！**


## 7.3 RTTI（runtime type identification）
编译器可能有的自己的一套内部类型体系，cfront实现的类型体系看上去有点像Java里的以Object作为所有对象的父类型类似。

对于函数的类型，有的gen和fct两种class，gen表示当前的函数是一个重载函数。

Downcast有潜在性的危险，因为它遏制了类型系统的作用，不正确的使用可能会带来错误的解释（如果它是一个read 操作）或腐蚀掉程序内存（如果它是一个write操作）。


如果一个指向gen object 的指针被不正确地转换为一个指向fct object 的指针pf。所有后续对pf的使用都是不正确的，这就是从RTTI的角度解释向下转型的危险性。

### 7.3.1 类型安全的向下转型
一个type-safe downcast必须在执行期对指针有所查询，看看它是否指向它所展现（表达）之object的真正类型。欲支持type-safe downcast，在object空间和执行时间上都需要一些额外负担：

1. 和java一样，需要额外的空间存储类型信息，通常是一个指针，指向类型信息节点

2. 需要额外的时间以决定执行期的类型（runtime type），因为，正如其名所示，这需要在执行期才能决定

为了取得RTTI功能的和效率的平衡，C++的RTTI机制提供了一个安全的downcast设备，但只对那些展现“多态（也就是使用继承和动态绑定）”的类型有效。

具体实现方法是：

所有polymorphic classes的objects都维护了一个指针（vptr），指向virtual function table。只要我们把与该class相关的RTTI object 地址放进virtual table 中（**通常放在第一个slot**），那么额外负担就降低为：每一个class object只多花费一个指针。这一指针只需被设定一次，它是被编译器静态设定的，而非在执行期由 class constructor设定（vptr才是这么设定的）

### 7.3.2 类型安全的动态类型转换
dynamic_cast运算符可以在执行期决定真正的类型。如果downcast是安全的（也就是说，如果base type pointer指向一个derived class object），这个运算符会传回被适当转换过的指针。如果downcast不是安全的，这个运算符会传回0。

所以，**dynamic_cast 转换为一定要先检查是否分为空**

结合前面的，类型信息被放在虚指针指向的数字的第一个槽，dynamic_cast可能的实现的：

```cpp
//关键的一步，取得当前实例的类型信息
((type_info*)(objetc->vptr[0]))->typr_descriptor;

//之后再交给下下面的步骤进行比较，看能不能进行转换

```

type_info是C++Standard所定义的类型描述器的class名称，该class中放置着待索求的类型信息。virtual table 的第一个 slot 内含 type_info object 的地址；此type_info object与pt所指的class type有关）。真实的类型和目标类型都被交给一个runtime library函数，比较之后告诉我们是否吻合。很显然这比static cast 昂贵得多，但却安全得多。


### 7.3.3 References并不是 Pointers
dynamic_cast可以用来动态转换指针，通过判断返回的指针的值是否为nullptr，确认是否是真实的类型。

用于转换引用时，由于引用的特殊性，不能的引用一个不存在的对象，转换成功时，没啥问题；如果转换失败，则会直接抛 bad_cast exception。

### 7.3.4 typeid运算符

typeid运算符传回一个**const reference**，类型为type_info。在先前测试中出现的equality（等号）运算符，其实是一个被overloaded的函数，如果两个type_info objects相等，这个equality运算符就传回true。

可见，typeid同样也可以实现类型鉴别的功能。

type_info的类定义在type_info.h头文件中，虽然RTTI提供的type_info对于exception handling的支持来说是必要的，但对于exception handling 的完整支持而言，还不够。如果再加上额外的一些type_info derived classes，就可以在exception发生时提供关于指针、函数、类等等的更详细信息。

虽然书中前面说过RTTI只适用于多态类（polymorphicclasses），事实上type_info objects 也适用于内建类型，以及非多态的使用者自定类型。这对于 exception handling的支持是有必要的。

对于非多态的场景，type_info object能在编译期就取得的，编译器就会在编译期直接进行运算，和constexpr差不多。

## 7.4 效率和便捷性
传统的 C++对象模型提供有效率的执行期支持。这份效率，再加上与 C 之间的兼容性，造成了C++的广泛被接受度。然而，在某些领域方面，像是动态共享函数库（dynamicallyshared libraries）、共享内存（shared memory）以及分布式对象（distributed object）方面，这个对象模型的弹性还是不够。

### 7.4.1 动态共享函数库（Dynamic SharedLibraries）
在目前的C++对象模型中，如果新版library中的class object 布局有所变更，上述的“library无侵略性”说法便有待商榷了，却在二进制层面（binary level）阻碍了弹性。

所以，直接看到阿里大佬推荐的"多态"写法，其实时为了保证二进制兼容性。


### 7.4.2 共享内存（Shared Memory）
当一个shared library 被加载时，它在内存中的位置由runtime linker 决定，一般而言与执行中的进程（process）无关。

然而，在C++对象模型中，**当一个动态的shared library支持一个class object（也就是在DLL中共享了一个全局的对象？？？），其中含有virtual functions（被放在shared memory中）时，上述说法便不正确。**

问题并不在于“将此object放置于shared memory中”的那个进程，而在于“想要经由这个shared object附着并调用一个virtual function”的第二个或更后继的进程。除非dynamic shared library被放置于完全相同的内存位置上，就像当初加载这个shared object的进程一样，否则virtual function会死得很难看，可能的错误包括segment fault 或bus error。

病灶在于每一个virtual function在virtual table中的位置已经被写死了。

目前的解决方法属于程序层面，程序员必须保证让跨越进程的shared libraries有相同的坐落地址（在SGI中，使用者可以根据所谓的so-location文件，指定每一个sharedlibrary的精确位置）。至于编译系统层面上的解决方法，势必得牺牲原本的virtual table实现模型所带来的高效率。