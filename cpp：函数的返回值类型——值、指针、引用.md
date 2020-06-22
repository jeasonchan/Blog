# 1 前言

参考文章：

C++函数返回引用       https://blog.csdn.net/qq_33266987/article/details/53516977

c++中，引用和指针的区别是什么？ - xmqv的回答 - 知乎      https://www.zhihu.com/question/37608201/answer/72766337


C++有三种传递方式：值传递、引用传递、指针传递

# 2 引用作为函数的返回值

函数返回值和返回引用是不同的！**函数返回值时会调用缺省的或者自定义的左值引用拷贝函数**产生一个临时变量作为函数返回值的副本(这是编译器偷偷实现的)，而**返回引用时不会产生值的副本**，既然是引用，那引用谁呢？这个问题必须清楚，否则将无法理解返回引用到底是个什么概念。以下是几种引用情况：

## 2.1 不可以返回局部对象的引用

```cpp
#include <string>
#include <iostream>

using namespace std;

string echo(string &input) {
    string result = input;
    return result;

};

//编译不过，直接报错，将一个局部变量作为左引用返回
//string &echo2(string &input) {
//    string result = input;
//    return result;
//
//};

string &&echo3(string &input) {
    const string &result = input;
    return const_cast<string &&>(result);
};


int main() {
    string input = "a";
    cout << echo(input) << endl;
//    cout << echo2(input) << endl;
    cout << echo3(input) << endl;


    return 0;
}

/*
//输出：

$ g++ -std=c++11 main2.cpp
main2.cpp: In function ‘std::string& echo2(std::string&)’:
main2.cpp:20:12: warning: reference to local variable ‘result’ returned [-Wreturn-local-addr]
   20 |     return result;
      |            ^~~~~~
main2.cpp:19:12: note: declared here
   19 |     string result = input;
      |            ^~~~~~
$ g++ -std=c++11 main2.cpp
$ ls
a.out  main2.cpp  main.cpp
$ ./a.out
a
a
 */
 ```

 上面的echo2函数，根本无法编译通过，由于局部变量的生命周期有限，且string类的析构函数必然会释放资源，所以只要返回值声明的是左值引用，返回到外面的必然是一个的指向无意义内存的引用。因此，绝不能返回局部变量的引用。

 指针同理，否则会返回一个野指针。

## 2.2 可以返回外部传入的引用

```cpp
const string &shorterString(const string &s1,const string &s2)
{
        return s1.size()<s2.size()?s1:s2;
}
```

以上函数的**返回值是引用类型。无论返回s1或是s2,调用函数和返回结果时，都没有拷贝这些string对象**。简单的说，返回的引用是函数的参数s1或s2，并且参数s1、s2也是引用，不是在函数体内产生的。函数体内局部对象是不能被引用的，因为函数调用完局部对象会被释放。

## 2.3 可以返回函数所属的对象的引用

作为类的成员函数，可以返回所在对象的引用，但是仍不能是函数内定义的类对象（会释放掉），一般为 this 指向的对象，典型的例子是 string类的赋值函数。

```cpp
String& String::operator =(const String &str)  //注意与“+”比较，函数为什么要用引用呢？a=b=c，可以做为左值  
{  
    if (this == &str)  
    {  
        return *this;    
    }  
    delete [] m_string;  
    int len = strlen(str.m_string);  
    m_string = new char[len+1];  
    strcpy(m_string,str.m_string);  
    return *this;  
}  
```
cpp既然也是面向对象，除了基础类型外，所有的对象传递也都要使用引用，除非极个别情况要使用值传递和指针传递。

## 2.4 可以返回this的成员变量的引用

```cpp
char &&a = '1';
a = '2';
cout << a << endl;

string b = "123";
string &c = b;

c="111";
//再一次证明，cpp里面的赋值符号不过是一个函数，这里相当于的调用了一个函数，入参是"111"，和java不太一样
//但是！！！string &c = b;  引用被声明时，所用的=是仅仅有绑定的意思的，和java保持一致
//但是，后来使用时就和普通变量名一样的


cout<<b<<endl;//输出 111


string d=b;//声明一个变量，并调用string的=函数，=函数是一个左值引用函数，应该是进行了深拷贝
d="456";
cout<<b<<endl;
```

**cpp的赋值符符号和java有很大的不同！！！java的=是引用绑定的意思，；在声明引用时，cpp的=同样仅用绑定对象的作用，但是，后来的=就是调用对象的“=”函数。**

# 3 引用和指针的不同

C++primer中对 对象的定义：

对象是指一块能存储数据并具有某种类型的内存空间一个对象a，它有值和地址&a，运行程序时，计算机会为该对象分配存储空间，来存储该对象的值，我们通过该对象的地址，来访问存储空间中的值

**指针p也是对象，它同样有地址&p和存储的值p**，只不过，p存储的数据类型是数据的地址。如果我们要以p中存储的数据为地址，来访问对象的值，则要在p前加解引用操作符"*",即\*p。

对象有常量（const）和变量之分，既然指针本身是对象，那么指**针所存储的地址**也有常量和变量之分，指针常量是指，指针这个对象所存储的地址是不可以改变的，而指向常量的指针的意思是，不能通过该指针来改变这个指针所指向的对象。

我们可以**把引用理解成变量的别名**。定义一个引用的时候，程序把该引用和它的初始值绑定在一起(声明引用时，=仅仅具有绑定地址的作用)，而不是拷贝它。计算机必须在声明r的同时就要对它初始化，并且，r一经声明，就不可以再和其它对象绑定在一起了，也就是不能换一个对象绑定。比如：

```cpp
int *pInt=1;
pInt=&2;//指针可以通过=绑定到一个新对象

//引用在声明并初始化之后，就成为了一个变量名，对其使用=也不只不过是调用类的=函数

```

实际上，**也可以把引用看做是通过一个常量指针来实现的，它只能绑定到初始化它的对象上**。

关于指针和引用的对比，可以参看<<more effective C++>>中的第一条条款，引用的一个优点是它一定不为空，因此相对于指针，它不用检查它所指对象是否为空，这增加了效率。比如下面的代码：

```cpp
int a,b,*p,&r=a;//正确，在栈用申请了4个变量并进行了初始化操作

int d=0；//现在栈中申请d并初始化0，也可能不初始化……再调用int的=函数

r=3;//正确：等价于a=3

int &rr;//出错：引用必须初始化

p=&a;//正确：p中存储a的地址，即p指向a

*p=4;//正确：p中存的是a的地址，对a所对应的存储空间存入值4

p=&b//正确：p可以多次赋值，p存储b的地址
```

最重要的一句话：引用可以理解成变量的别名，以把引用看做是通过一个常量指针来实现的。

# 4 总结
函数的入参和返回值，**都可以通过声明为引用类型从而避免语义上告诉编译器要进行值传递**，从而避免值传递的拷贝构造和拷贝赋值的性能问题
