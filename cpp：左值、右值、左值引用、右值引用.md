# 1 前言

参考文章：

C++ 11 左值，右值，左值引用，右值引用，std::move, std::foward     https://blog.csdn.net/xiaolewennofollow/article/details/52559306

在C++11中，左值、右值、左值引用、右值引用的概念开始比在之前的版本中提高到很重要的地位，充分使用可以用cpp实现更方便、安全的内存管理的编码。

# 2 左值和右值的定义

左值和右值在C中就存在，不过存在感不高，在C++尤其是C++11中这两个概念比较重要，左值就是有名字的变量（对象），可以被赋值，可以在多条语句中使用;而右值呢，就是临时变量（对象），没有名字，**只能在一条语句中出现**，不能被赋值。

在 C++11 之前，右值是不能被引用的，最大限度就是用**常量引用**绑定一个右值，如 :

```cpp
//各种常量其实就是临时变量
const int& i = 3;

//调用了string的赋值符号("=")进行赋值操作
//"123"是作为const char* 类型的入参
string str="123";
```

在这种情况下，右值不能被修改的，因为是一个没有姓名的对像。但是实际上右值是可以被修改的，如 :

```cpp
T("123").set().get();
```

T 是一个类，先用有参构造函数产生了一个右值对像；set 是一个函数为 T 中的一个变量赋值，get 用来取出这个变量的值。在这句中，T() 生成一个临时对象，就是右值，set() 修改了变量的值，也就修改了这个右值。

**既然右值可以被修改，那么就可以实现右值引用。右值引用能够方便地解决实际工程中的问题，实现非常有吸引力的解决方案。**

# 3 右值引用

左值的声明符号为"&"， 为了和左值区分，右值的声明符号为"&&"，是两种不同的类型。

给出一个实例程序如下：

```cpp
#include <iostream>

void process_value(int& i) 
{ 
  std::cout << "LValue processed: " << i << std::endl; 
} 

void process_value(int&& i) 
{ 
  std::cout << "RValue processed: " << i << std::endl; 
} 

int main() 
{ 
  int a = 0; 
  process_value(a);
  process_value(1); 
}
```

运行一下：

```
wxl@dev:~$ g++ -std=c++11  test.cpp
wxl@dev:~$ ./a.out 
LValue processed: 0
RValue processed: 1
```

Process_value 函数被重载，分别接受左值和右值。由输出结果可以看出，临时对象是作为右值处理的。

那么，问题来了。如果一个显式声明的右值引用的变量指向了一个右值，调用的时候是按左值引用还是右值引用？事实说话：

```cpp
#include <iostream>

void process_value(int& i) 
{ 
  std::cout << "LValue processed: " << i << std::endl; 
} 

void process_value(int&& i) 
{ 
  std::cout << "RValue processed: "  << std::endl; 
} 

int main() 
{ 
  int a = 0; 
  process_value(a);
  
  //int& haha=3;  这一句会编译错误，因为左值引用无法绑定到的一个右值

  //将右值引用绑定到右值
  int&& x = 3;
  process_value(x); 
}
```

尝试运行：

```
wxl@dev:~$ g++ -std=c++11  test.cpp
wxl@dev:~$ ./a.out 
LValue processed: 0
LValue processed: 3
```

x 是一个右值引用，绑定到一个右值3，但是**由于x是有名字的，所以x在这里被视为一个左值，所以在函数重载的时候选择为第一个函数**。

# 4 右值引用的意义
直观意义：为临时变量续命，也就是为右值续命，因为右值在表达式结束后就消亡了，因为毕竟是一个局部变量。在不使用右值引用的情况下，如果想继续使用右值，那就会动用昂贵的拷贝构造函数。（关于这部分，推荐一本书《深入理解C++11》）

右值引用是用来支持**转移语义**的。

转移语义可以将资源 ( 堆、系统对象等 ) 从一个对象转移到另一个对象，这样能够减少不必要的临时对象的创建、拷贝以及销毁，能够大幅度提高 C++ 应用程序的性能。临时对象的维护 ( 创建和销毁 ) 对性能有严重影响。

转移语义是和拷贝语义相对的，可以类比文件的剪切与拷贝，当我们将文件从一个目录拷贝到另一个目录时，速度比剪切慢很多。通过转移语义，临时对象中的资源能够转移其它的对象里。

在现有的 C++ 机制中，我们**可以定义拷贝构造函数和赋值函数**。要**实现转移语义，需要定义转移构造函数，还可以定义转移赋值操作符**。对于右值的拷贝和赋值会调用转移构造函数和转移赋值操作符。**如果转移构造函数和转移赋值操作符没有定义，那么就遵循现有的机制，直接针对左值的 拷贝构造函数  和  赋值操作符。**

普通的函数和操作符也可以利用右值引用操作符实现转移语义。

下面介绍定义转移构造函数和转移赋值运算符。

## 4.1 转移语义、转移构造函数、转移赋值运算符

以string为例，实现拷贝构造函数和赋值操作符，然后再实现  右值引用构造函数  和  右值引用赋值操作符。

```cpp
 class MyString { 

 private: 
  char* _data; 
  size_t   _len; 

  void _init_data(const char *s) { 
    _data = new char[_len+1]; 
    memcpy(_data, s, _len); 
    _data[_len] = '\0'; 
  } 

 public: 
  MyString() { 
    _data = nullptr; 
    _len = 0; 
  } 

  MyString(const char* p) { 
    _len = strlen (p); 
    _init_data(p); 
  } 

//左值引用  拷贝构造函数
  MyString(const MyString& str) { 

    //和java一样，同一个域/类 内部，可以访问类实例的私有变量
    //之前一直认为私有变量只能通过this和getter才能访问……
    _len = str._len; 
    _init_data(str._data); 
    std::cout << "Copy Constructor is called! source: " << str._data << std::endl; 
  } 


//左值引用  的 赋值  符号
  MyString& operator=(const MyString& str) { 
    if (this != &str) { 
      _len = str._len; //为了让两个对象脱离关系，这里的值拷贝是必不可少的，即不能共用内存块
      _init_data(str._data); //因为MyString和当前的类类型相同，所以可以直接访问入参的私有变量
    } 
    std::cout << "Copy Assignment is called! source: " << str._data << std::endl; 
    return *this; 
  } 

  virtual ~MyString() { 
    if (_data) delete(_data); 
  } 

 }; //类定义结束

 int main() { 
  MyString a; 
  //在栈中声明变量a，并直接调用无参构造函数在栈中进行初始化，然后将a绑定到栈中刚刚初始化好的内存区域

  a = MyString("Hello"); 
  //先调用有参构产生一个MyString类型的右值
  //再通过该类的重载的赋值符号进行赋值操作，所以一定会打印：
  //Copy Assignment is called! source: Hello
  //注意了！！！！赋值符号重载只针对了左值引用进行重载，但是找不到右值引用时就直接用左值引用的

  std::vector<MyString> vec; //声明并调用无参构造

  vec.push_back(MyString("World")); 
  //先调用有参构产生一个MyString类型的右值
  //然后调用vector的
  //void  push_back(value_type&& __x)   方法，入参是右值引用
  //这个方法内部会将真实的如参转换为右值，由于没有转移拷贝函数，只能直接使用针对左值的拷贝构造函数
 }
 ```

运行一下：

```bash
jeasconchan@jeasconchan-ThinkPad-E485:~/projects/CppExercise/CppNewStarter/试验左值右值$ g++ main.cpp 
jeasconchan@jeasconchan-ThinkPad-E485:~/projects/CppExercise/CppNewStarter/试验左值右值$ ls
a.out  main.cpp
jeasconchan@jeasconchan-ThinkPad-E485:~/projects/CppExercise/CppNewStarter/试验左值右值$ ./a.out 
Copy Assignment is called! source: Hello
Copy Constructor is called! source: World

```

先强调一下，cpp的类中，自带四个默认方法：无参构造、无参析构、拷贝沟在函数、拷贝赋值函数，默认的拷贝构造和拷贝赋值都是全量拷贝的，使用起来会有比较大的开销。尤其是，**对象如果是个右值，却没有定义转移构造和转移赋值函数时，会直接使用左值的拷贝构造和拷贝赋值。**

为什么要有拷贝构造和拷贝赋值？

因为，我们想把新对象 和 被拷贝的对象和被用来赋值的对象  **完全分离开来，不希望有任何公用的内存片段**。

为什么要有转移构造和转移赋值？

因为，**从函数中返回的对象是我们希望直接使用的**，并不希望中间多加拷贝构造和拷贝赋值来这些没用的步骤，而且按照道理，离开作用域之后，函数中产生的对象应该是被析构函数释放调的，所以，我们要对右值的引用重载  转移构造函数和转移赋值操作符。

先定义转移构造函数：

```cpp
  MyString(MyString&& str) { 
    std::cout << "Move Constructor is called! source: " << str._data << std::endl; 
    _len = str._len; 
    _data = str._data; //注意此处，没用左值引用的 memcpy ，而是直接用的指针指向原来的地址
    
    str._len = 0; 
    
    str._data = nullptr; 
    //将原先的指针赋值为空很关键，观察析构函数：
    //virtual ~MyString() { 
    //if (_data) delete(_data); 
    //} 
    //指针不为空释放堆内存，如果不赋值为nullptr，内存会被释放，
    //转移拷贝的里面指向的内存区域就没有意义了，或者很快重新写入其他数据


 }
 ```

总结一下转移拷贝的要点，及右值引用可拷贝函数的要点：
1. 务必明白右值引用拷贝函数的目的：代替原来的拷贝构造，提高效率
2. 参数（右值）的符号必须是右值引用符号，即“&&”。传参的时候还是直接右值。
3. 参数（右值）不可以是常量，因为我们需要修改右值。比如，某些属性赋值为nullptr
4. 参数（右值）的资源链接和标记必须修改。否则，**右值的析构函数就会释放资源。转移到新对象的资源也就无效了**。

再来定义转移赋值函数：

```cpp
//函数的返回值是左值引用
MyString& operator=(MyString&& str) { 
  std::cout << "Move Assignment is called! source: " << str._data << std::endl; 
  if (this != &str) { //引用就像一个对象，用了取地址符
    _len = str._len; 
    _data = str._data; 
    str._len = 0; 
    str._data = NULL; 
  } 
  return *this; //返回的是一个对象，用了取内容符号
}
```

有了右值引用和转移语义，我们在设计和实现类时，对于需要动态申请大量资源的类，应该设计转移构造函数和转移赋值函数，以提高应用程序的效率。

# 5 td::move()和std::forward