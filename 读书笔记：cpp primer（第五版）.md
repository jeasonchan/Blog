# 1 开始

```cpp
#include <iostream>

using namespace std;

int main()
{
    cout << "标准输出" << endl;
    cerr << "标准错误" << endl;
    clog << "运行时消息" << endl;

    cout << "输入两个数字：" << endl;

    int number1;
    int number2; //

    cout << "初始化为：" << number1 << " " << number2 << endl;

    cin >> number1 >> number2; //一直会阻塞直到输入了两个非空的字符

    cout << "number1:" << number1 << "  number2:" << number2 << " and sum is " << number1 + number2 << endl;

    cout << "/*" << endl;
    cout << "/*" << endl;

    // while语句
    int sum = 0, val = 1;
    while (val <= 10)
    {
        sum += val;
        val++;
    }
    cout << "sum=" << sum << endl;

    bool haha = true;

    sum = 0;
    val = 10;

    while (val >= 1)
    {
        sum += val;
        --val;
    }

    sum = 0;
    val = 50;
    for (; val > 0; ++val)
    {
        sum += val;
    }

    sum = 0;
    val = 0;
    //while的判断条件是 cin>>val  这个表达式的值
    //也就是  >>  这个函数的值，也就是标准输入流  cin
    //当cin遇到 文件结束符（eof） 或者 无效输入时，cin的状态为无效状态，即为false
    //其与可以人为时有效的
    while (cin >> val)
    {
        cout << "Got:" << val << endl;
        sum += val;
    }

    //想推出以上循环，就只能故意输进去字符，强制cin无效；或者，ctrl+z然后enter，输入文件结束符

    cout << "end and sum is " << sum << endl;

    return -1;
}
```

未初始化的变量：

基本数据类型声明时，确实会有一个初始值，但是，是不确定的初始值。类 类型的变量如果未指定初始值，就会自动调用无参构造（可能时编译器自动添加的无参构造）。类类型的成员变量，在为对象申请内存时就会按照类成员定义的先进行一波初始化，**然后才会开始**调用构造函数，所以，为了避免在构造函数中**二次初始化**自定义类型的成员变量，引入了参数列表。

```cpp
private:
    string s_;

public:
Base(const string& s) 
{ 
    s_.string::string(s); 
} 

/* 
真是的执行顺序是：

string s_;//在构造函数之前先执行

Base(const string& s) { 
    s_=s;//这是一个左值赋值函数，会不会有较大的性能开销看string的=实现
} 
*/

```

# 2 变量和基本类型
## 2.1 内置的基本类型
cpp和java一样，虽然都是面向对象的，本应该是一切皆对象（cpp其实可以多种，过程、对象、函数式，都可以），但还是有的旧时的影子，那就是基本数据类型。

cpp的基本数据类型，分为算术类型和空类型（void）。

### 2.1.1 算数型
算数型还分为：
* 整型，包括 bool、char、wchar_t、char16_t char32_t、short、int、long long、long
* 浮点型，包括 float、double、long double

整型之所以叫整型，不仅体现在代表一个整数，在内存中，递增是通过向低位+1实现的，而浮点型是**个人猜测**，会像java一样，某些位是指数位。

每种类型的占用的比特位数稍有不同。其中最坑的莫过于字符相关的整型：char、wchar_t、char16_t、char32_t,稍有不慎就会发生截断，造成字符显示异常。

C++读写汉字，C++处理中文字符             https://blog.csdn.net/calmreason/article/details/7935258

C++11新特性--Unicode 16位和32位支持           https://www.jianshu.com/p/8b87a05c23b1

浅谈char类型范围（原码、反码、补码）       https://blog.csdn.net/zy986718042/article/details/71699079


内置类型的机器实现：

<!-- 补充实现 -->

可寻址的最小内存块  字节，现代操作系统固定为8比特（1比特=1位）

存储的基本单元  字，4字节或者8字节

整型中除了布尔型，还可以分为：有符号型和无符号型，默认的int、long、long long是有符号的。

比较特殊的是char、signed char、unsigned char，尽管看上去是三种，但是，**因编译器实现而异，char必然是unsigned char和signed char其中的一种**。unsigned char能表示0到255，signed char能表示-128到127。不过，ASCII一共只有128个字符，无符号和有符号的char会用非负数表示ASCII中的字符。

由于cpp就是为了性能而生，且为了合理、充分利用硬件，cpp的基本数据类型有很多种，所以**如何选择类型？选择数据类型的标准是什么？**

1. 明确知道类型为非负时，使用unsigned的数据类型
2. 一般使用int。int不够用时，建议直接使用long long，因为long一般和int有一样的位数
3. 不要char或者bool进行算数计算，以为你char的有无符号取决于编译器实现，实在想用务必显式声明signed char或者 unsigned char
4. 浮点数运算建议直接使用double。因为，因为float消耗并没有比double少多少，甚至有时候float还比double慢

### 2.1.2 类型转换
一个对象的定义，包含了的该对象包含的数据，还包含了能参与的运算/方法，其中有一种运算被大多数数据类型所支持，那就是类型转换。类型转换，是指，当某个日地方的参数因该是某种类型，但是我们却给定了另一种（能进行转换的）类型，则**编译器会自动将我们使用的数据类型自动转换为入参期望的数据类型**。Java下也有一样的特性，java支持低精度往高精度转，而高精度向低精度转这种发生数据截断的情况会报错。

cpp中的类型转换会在以下2种情况发生：

1. 显式的赋值语
```cpp
bool b=42;//b为真，数字类型向布尔转换，0转成false，其余转为true
int i=b;//布尔值向数字类型转换，true转为1，false转为0
i=3.14;//i被截断为3
double pi=3;//pi是3.0
unsigned char c=-1;//假设char是8bit的，则c是对2^8取模的余数，即为256-1=255
signed char c2=256;//假设char是8bit的，由于cpp并不规定有符号数的具体实现，无法确定的最终的值，c2为未定义
```

2. 当程序中使用一种算术类型，但是，实际上需要另一种类型时，编译器会自动执行类型转换
```cpp
int i=666;
/*
在if的判断条件中，i会自动转为true
*/
if(i){
    //xxxxxxx
}
```
个人觉，还有种其中，比如的某个类的构造函数的入参就只有一个算术类型，使用赋值语句时，有可能自动调用构造函数，详见explict关键字用法



#### 2.1.2.1 无符号类型注意点
对于无符号类型，超过范围的数都会通过取模运算自动转换到范围内，且不会报错，极有可能的造成死循环。

无符号数之间进行运算时，务必要提前判断结果是否会超出范围而发生取模运算。

同时，有符号和无符号进行运算时，**有符号首先被转换成无符号**，在进行无符号数之间的运算，无符号数之间的**运算结果还有可能超出无符号数的范围，再次发生取模运算**。由于这个操作比较复杂，所以**绝对不要进行**无符号和有符号的混合运算。

### 2.1.3 字面值常量
一看就知道值的常量就是字面值常量，字面值常量的形式和值的大小决定了该字面值常量的数据类型。

#### 2.1.3.1 整型和浮点型字面值常量
对于十进制整型，如:123，44444444444444444444444444，在能容纳字面值常量的前提下按照int、long、long long先后顺序当做字面值常量的类型。

对于八进制（比如，024）、十六进制（比如，0x14）也有类型的数据类型顺序。

尽管整型字面值可以存储在有符号类型中，但是，**严格来说十进制字面值不会是负数**，-42这种只是对十进制字面值常量取负值而已。

#### 2.1.3.2 字符和字符串字面值

```cpp
'a';//由单引号括起来的一个字符被称为char型字面值
"2333333";//由双引号括起来的两个或者多个字符则构成字符串型字面值
```

字符型字面值就真的表示一个字符，字符串型字面值的长度会被看到的长度多1，因为，字符串字面值常量的本质是char型数组，且末尾多一个字符'\0'。

如果两个字符串字面值的位置紧邻，且之间仅有空格、缩进、换行符分隔，则这两个紧邻的字符串常量是一个整体，这种写法常用来解决输出比较长而换行写的情况。如：

```cpp
std::cout<<"22222222222222222222"
            "2222222222222222222222222222"<<std::endl;
```
##### 2.1.3.2.1 转译序列
有两类字符程序员无法**直接使用**：
1. 不可打印的字符，比如退格等控制字符，因为它们没有可视的图符
2. 特殊含义的字符，比如：单引号、双引号、问好、反斜线，

在以上情况下，要想正常使用则必须使用反斜线进行转译，才能当做正常的字符使用。

```
换行符   \n
反斜线   \\
回车符   \r
退格符   \b
问号     \?
报警     \a
双引号   \'
单引号   \"
```
同时，也可以借助转译符号，通过 转译符号+数字 的方式直接使用字符集中的字符。有以下两种使用发：

1. \x后面紧跟一个或者多个十六进制数组，以表示一个或者多个字符

2. \后面跟1个、2个、3个八进制数字，以表示一个字符；注意，如果反斜杠后面的八进制数超过三个，转义符号只能转译前三个。

比如：

```
\115   字母M，3个八进制数
\40    空格，2个八进制数
\0     空字符，1个八进制数
\x4d   字符M，1个十六进制数
\x4dO\115    表示字符串MOM
\1154       表示字符串 M4
\0x1231231231231231  就表示
```
#### 2.1.3.3 显式声明字面量常量的类型
如果不想让编译器自动推断字面量常量的类型，可以显示声明字面量常量的类型信息，比如：

```
L'a'   声明是wchar_t类型的字符型常量
u8"hi!"   声明是utf8编码的字符串字面值
42ULL    声明是unsigned long long，long一定要使用L大写来表示以避免小写的L和数字1混淆
1E-3F    声明是float
3.14159L    声明是long double
```

#### 2.1.3.4 布尔值的字面值和空指针字面值
```cpp
bool a=true;
a=false;
a=nullptr;
```

## 2.2 变量
变量提供一个具名的、可供程序操作的存储空间。cpp中每个变量都有其数据类型，数据类型决定着：

1、变量所占空间的大小和布局方式

2、变量所占用的空间可能存储的值的范围

3、变量能参与的运算

cpp中，变量和对象的概念一般是可以互换使用的。

### 2.2.1 变量定义
变量的定义的基本形式：类型说明符+1个或者多个变量名组成列表+分号。列表中每个的变量名的类型都是由类型说明符指定，定义的同事还可以顺带为变量赋初始值（也就是，进行显式初始化）。

比如：

```cpp
int sum=0,
    value,
    units_sold=0;//三个有符号整形，其中两个初始化为0

SalesItem salesItem;//SalesItem类型的变量，按照改类中定义无参股构造进行初始化

std::string book(L"c++ primer");//book通过string的有参构造和一个声明为宽字符型的字符串字面值常量进行初始化
```
上面的代码示例中，使用了字符型常量来初始化stirng对象，本质就是将字面值拷贝给了string对象。


#### 2.2.1.1 初始值
对象在创建时就立刻获得了特定的值，称这个对象被初始化了。一般情况下，对象创建的时刻，是拥有名字的那个时刻，就算是在同一条语句中，先初始化的变量可以用于其他变量的初始化，但是，好像并不建议这样。比如：


```cpp
// price先被定义并赋值，然后用于初始化total
double price=100.00,total=10.0*price;
```

cpp中，初始化是个异常复杂的问题。

先纠正常识性误解：因为可以用赋值符号（=）来初始化变量，很多人就认为初始化、赋值的一些关系，比如，误认为初始化是赋值的一种。

**实际上，cpp中，初始化和赋值是两种完全不同的操作。尽管两者的区别很小很小，但是，两者的概念上的区别还是很大的，而且至关重要！初始化不是赋值，初始化的含义是含义是创建变量时赋予其一个初始值，而赋值的含义是把对象的当前值擦除，并且用新值代替。**


#### 2.2.1.2 列表初始化
cpp的初始化有好几种方式，比如：

```cpp
int age=0;
int age={0};
int age(0);
int age{0}
```

作为cpp11标准的一部分，用花括号初始化变量得到了全面应用，之后的章节会详细介绍**列表初始化**，现在，**无论是初始化对象还是为对象赋值新值，都可以使用一组由花括号括起来的初始值了**。

通知，使用花括号的时，如果值的精度发生丢失，则会直接报错，而传统的圆括号则不会。比如：

```cpp
long long id=3.14159;
int a{id},b={id};// 报错，精度丢失
int c(id),d=id;//执行，自动发生截断，丢失一部分精度
```

#### 2.2.1.3 默认初始化
如果定义变量时，没有指定初始值，则变量会被**默认初始化**，变量会被赋予**默认值**。

默认值由变量所在的位置和其类型共同决定。


对于cpp的内置类型，默认值由变量的位置决定。如果该变量在一个函数（包括main函数）体中，变量定义时未显式初始化，则申请的内存中的比特位是混乱的，随便读写会引发错误，也就是：

函数体中的内置类型变量不会自动初始化，函数体外（比如全局的变量）会根据类型自动隐式初始化，比如，int默认为0，string默认为"\0"。

根据深入探索C++对象模型和代码实践，编译器自动生成的构造函数不会对类中的基础数据类型的成员成员变量进行0值初始化。基础数据类型作为类成员变量时，C和CPP的编译器都不会主动对其进行0值初始化，对基础数据类型的显示初始化必须有程序员自己进行。编译器只负责调用类的默认构造函数。

对于自定义的类型，默认值由该类型的无参构造的执行体中语句决定。

最佳实践：**自己显式初始化每个内置类型的变量**，虽然没有必要这么做，但是，这样做是绝对简单可靠。

### 2.2.2 变量声明和定义的关系
为了支持分离式编译，cpp将声明和定义区分开来。声明时的名字被程序所知，定义负责创建和名字相关的实体。

变量可以被声明多次，但只能被定义一次。如果只想声明而不定义变量，使用extern关键字，如：

```cpp
extern int i;//只声明i
int j;//声明、定义、初始化i（假设此行代码直接写在函数体外）

extern int k=123;//声明、定义、初始化一个变量
```

声明和定义的区别看起来好像微不足道，但实际上却非常重要。如果要在多个文件中使用同一个变量，比如Pi，就必须将声明和定义分离。假设Pi的定义写在Math.h文件中，则对Pi的定义只能出现在一个源文件中，对于地方对Pi使用，都必须先用  extern double Pi  声明一下，之后才能通过Pi这个变量名对其值进行访问。

### 2.2.3 标志符
也即是变量名，变量名的命名规范和java类似


### 2.2.4 名字的作用域
作用域（scope）是程序的一部分，cpp中作用域大多以花括号进行划分。和Java不同的是，cpp中可以直接在类的外部、头文件、源文件中声明/定义方法、变量，这些直接在类外声明/定义的方法、变量拥有**全局作用域**。

```cpp
#include <iostream>

int main(){
    int sum=0;
    for(int i=0;i<100;++i){
        sum+=i;
    }
}

std::cout<<"sum="<<sum<<std::endl;

return 0;
```

这段程序定义了三个名字，main、sum、i，其中，命名空间std提供了cout和endl；main直接定义在所有花括号之外，拥有全局作用域；sum和i就和java中的局部变量一样。

最佳实践：第一次使用时再定义变量，定义和使用距离太远，代码可读性会变差。

内层作用域中定义了外层作用域中的同名变量时，不明确具体是哪个的情况下，取内层定义的值使用，比如：

```cpp
#include <iostream>

int age=123;

int main(){
    int age=666;
    std::cout<<age<<std::endl;//输出666
    std::cout<<::age<<std::endl;//输出123，通过域操作符使用了全局变量的age
    return 0;
}
```

## 2.3 复合类型
复合类型（compound type）是指基于其他类型定义的类型。cpp的语言中有几种复合类型，此处只介绍两种：引用和指针。

和复合类型相对的是基本类型（base type），cpp内置数据类型、自定义的结构体、类其实都是基本类型的范畴。

### 2.3.1 引用
cpp11开始，新增了一种右值引用（rvalue reference），博客中之前也探索过，各种字面量其实都是右值引用。

平时说的引用，无特殊说明时，都是指左值引用（lvalue reference）。

cpp的引用和java里的引用几乎一毛一样，本质上可理解为一个常指针，**定义的时候必须显式进行初始化。**

```cpp
int value=1024;
int &refVal_1=value;
int &refeVal_2;//报错，编译都不过，引用定义时必须初始化
```

引用定义是就必须初始化，也就是，引用定义时就必须将其和指向的真正的值进行绑定（bind），也就是将数据指针指向真实的对象，且无法更改绑定关系。

**引用并非对象，只不过是为已存在的对象起了另外一个名字。因此，在代码层面上，引用变量名和原始变量名可完全相互替换。**

```cpp
int i=1024;//定义、初始化一个的int变量
int &ref_i=i;//定义、初始化一个引用，给i起一个别名
int $ref2_i;//报错，应用必须显式初始化

ref_i=3;
//与i=3的效果一模一样，月java略有不同，java中的引用这样赋值会指向新的对象，旧的的对象还继续存在。cpp这样赋值，是擦除旧值，用新值代替，就对象发生了变化。
```

### 2.3.2 指针
和引用类似，指针也实现对其他对象的间接访问。但是，和引用又有两点不同：

1. 指针本身就是一个对象，允许对指针进行拷贝、赋值，且的指针对象的生命周期中，可以先后指向不同的对象
2. 指针无需在定义时赋值，和其他内置类型一样，在函数中，如果定义时没有显示初始化，则会有一个随机的初始值，对该野指针乱操作甚至可能导致系统崩溃

```cpp
int i,*pi;
//假设在函数内部这么定义，则定义了整型变量i，i目前是一个随机值
//整型指针pi，pi指向了一个随机的地址

//如前文所说，在函数内部还是显式初始化安全一点
int i2=0,*pi2=nullptr;
```

#### 2.3.2.1 获取对象的地址
指针存放了某个对象的地址，要想获取对象的地址，需要使用取地址符&：
```cpp
int value=233;
int *pValue=&value;//定义指针变量pValue，并初始化为value的地址
```
注意，引用不是对象，指针是对象，所以，**引用没有实际地址，也就无法定义指向引用的指针。**但是，可以对引用进行取地址操作，取到的时绑定的对象的地址，如：

```cpp
int value = 123;
int &rValue = value;
int *pValue = nullptr;
pValue = &value;
cout << "pValue = &value;  pValue:" << *pValue << endl;//输出123

pValue = &rValue;
cout << "pValue = &rValue;  pValue:" << *pValue << endl;//输出123
```

可见，目前来看，引用和变量名在代码层面是完全等价的。

#### 2.3.2.2 指针的值

任何时候，指针的值必定是以下几种情况之一：

1. 指向一个对象
2. 空指针，nullptr，没有指向任何对象
3. 指向紧邻对象所占空间的下一个位置
4. 无效指针

情况3和4直接读、写、解引用指针都会引发错误，只有1和2还算比较可控

#### 2.3.2.3 利用指针访问对象

使用解引用符号\*来访问指针指向的对象，可以说，\*p、原始的变量名、原始变量的引用 在代码层面上是完全等价的。

解引用仅适用于确实指向了某个对象的指针,对空指针和无效指针解引用会直接跑异常中断线程，而且在Linux下好像也没法捕捉，因为：

Dereferencing null pointers, division by zero etc. does not generate exceptions in C++, it produces undefined behavior.

只有在windows下使用SEH才能捕捉系统异常，Linux下都需要自己事先判断。

```cpp
int value=123;
int *pValue=&value;

cout<<*pValue<<endl;//解引用，输出123

*pValue=0;//通过指针直接操作对象
cout<<*value<<endl;//输出0
```

#### 2.3.2.4 &和\*的多重含义
&和\*这样的符号，既能作为表达式里的运算符，也能作为声明/定义的一部分出现，符号出现的上线文决定了符号的含义：

```cpp
int value=42;
int &rValue=value;//&紧跟类型说明符出现，是声明的一部分，起到声明引用的作用
int *pValue=nullptr;//*紧跟类型说明符出现，是声明的一部分，起到声明指针的作用

pValue=&value;//&出现在表达式中，是取地址符
*pValue=123;//*出现在表达式中，是解引用符
```

#### 2.3.2.5 空指针

空指针不指向任何对象，在试图使用一个指针（解引用操作等）之前，应该先检查是否为空。定义指针时，为了不必要的麻烦，一开始没有值最好就初始化为空指针，空指针的的生成方法以有：

```cpp
// C++11中出现的新字面量，
// 该字面量可以被转换成其他任意类型的指针
int *p1=nullptr;

// 将指针对象的值，初始化为0
// 也就是地址的值是0
// 使用解引用的符号对其就引用就是对0地址进行解引用
// 百分百引抛出系统异常
int *p2=0;

//cstdlib头文件中定义的值，就是0，
// 所以，等价于int *p2=0;
int *p3=NULL;
```

上面的NULL，是一个预处理变量，在预处理阶段由预处理器自动替换为设定的值。新标准下，建议直接使用nullptr。

对以上三种方法得到的空指针进行解引用操作都会报系统异常。因此，**一定要显式初始化所有的指针，并且，尽可能的先定义对象，在定义指向对象的指针**，因为，解引用空指针是系统异常，无法直接用捕捉，报错了也看不到堆栈信息。

#### 2.3.2.6 赋值和指针
指针和引用都是对对象间接访问的方式。

引用再定义时必须初始化，且无法再绑定到的新的对象；引用只是一个符号。

指针在定义时不必初始化（但是，建议初始化为nullptr），可以指向任意新的同类型对象。指针本身就是一个对象，指针的值是对象的地址。

```cpp
int value=123;
int &rValue=value;
int *pValue=&rValue;
```
"*pValue"、"rValue"、"value"，这三个表达式，在代码层面上是完全等价的。

#### 2.3.2.7 其他指针操作
两个指针变量可以使用==和!=来进行比较，比较的内容是指针对象里存的地址，地址相同则为true，反之为false。

注意，==结果为true有三种情况：

1. 操作符两边的指针都是空指针
2. 都指向同一对象 或者 同一对象的下一地址
3. 一个指向A对象，另一个人指向B对象的下一个对象（恰好是A）的地址

#### 2.3.2.8 void *指针
可以存放任意对象的地址，但是，通过void\*指针能做到事情比较有限：

1. 和别的指针比较
2. 作为函数的输入和输出
3. 赋值给另外的指针

**无法通过void*指针访问、修改所指向的对象，因为，不知道的所指向的对象类型。**

```cpp
double obj=3.14,*pDouble=&obj;
void *pObj=&obj;
pObj=pDouble;

// 下面这一句编译报错：
// error: 'void*' is not a pointer-to-object type
cout << "*pObj:" << *pObj << endl;

// 要想使用必须先转型成具体的指针类型，再使用：
double *pDouble2 = (double *)pObj;
cout << "*pDouble2 :" << *pDouble2 << endl;
```


### 2.3.3 理解复合类型的声明

#### 2.3.3.1 指向指针的指针

```cpp
int value = 1024;
int *pValue = &value;
int **ppValue = &pValue;

cout << value << endl;//1024
cout << *pValue << endl;//1024
cout << *ppValue << endl;//0x22fd8c
cout << **ppValue << endl;//1024
```

#### 2.3.3.2 指向指针的引用

```cpp
int value = 1025;
int *pValue = &value;

// pValue是一个地址，所以是 int * xxxxx =pValue
// 又因为xxx是引用，所以用 &rpValue 声明
// 最终如下：
int *&rpValue = pValue;

cout << *pValue << endl;  //1025
cout << *rpValue << endl; //1025
```

**要明白rpValue究竟是什么类型，最简单的办法是从右往左阅读**"int *&rpValue"：

1. rpValue是变量名
2. &rpValue 表明是一个引用
3. \*（rpValue）表明是一个指针的引用，即为，一个指针的别名

## 2.4 const限定符
有时候我们希望变量的值在运行时不能改变，因为，这个值在运行时改变可能会产生BUG，比如：传输协议中某N个字节代表的含义，因为是协议，所以这个N绝对不能改变。

为了避免在运行时某变量被修改，可以用**关键字const对变量的类型加以限定**，是对变量类型限定，不是对变量加以限定。const类型的变量一旦创建后就无法就修改，所以，**const类型的定义时就必须进行初始化，初始值可以是任意复杂的表达式**。

```cpp
const int buffrtSize=512;
//任何试图修改的行为都将引发错误，并且编译检查都过不了
```

const修饰基本数据类型，基本数据类的值肯定是不能修改的了，毋庸置疑。**用const修改对象类型时，该对象就只能访问被 const 修饰的成员了（包括 const 成员变量和 const 成员函数）**，因为非 const 成员可能会修改对象的数据（编译器也会这样假设），C++禁止这样做。

**初始化和const**

对象的类型决定了该对象的所支持的操作，const修饰了类型，可以说，const typeName 组成了在一种新的类型，那这种新类型（const typeName）和原始的typeName所支持的操作有什么区别？

对于基本数据类型，读操作完全没有区别（可以读出来进行各种运算、用来初始化等等），不支持写操作。

**const对象默认只在当前文件内有效**
想在文件中直接定义（而不是作为class的成员变量）一个变量，并被其他文件使用，需要这样：

1. 方法一

file1.h
```cpp
// 必须用namespace包一下，要不然，包含了file1.h文件的文件中，
// 就不能定义变量a,对别人造成变量名污染。
// 如果使用了a，就会报重复定义a的编译错误
namespace jeason{
    int a=123;
} 
```

别的文件使用：
```cpp
inlclude "file1.h"

// 仅能声明，cpp可多次声明，单次定义
extern int jeason::a;

```


2. 方法二

file1.h
```cpp
namespace jeason{
    extern int a;
}
```

file1.cpp
```cpp
inlclude "file1.h"

// 正式定义
int jeason::a=123;
```

别的文件使用：
```cpp
inlclude "file1.h"

extern int jeason::a;
```

对const类型的变量，好像也能按照上面的方法定义，并被使用，但是！！！！！const被设定仅在当前文件中有效，因此，相比于普通变量，定义时需要再额外加一个extern，即为：

1. 方法一

file1.h
```cpp
namespace jeason{
    const int a=123;
} 
```

别的文件使用：
```cpp
inlclude "file1.h"

extern const int jeason::a;

```

2. 方法二

file1.h
```cpp
namespace jeason{
    extern const int a;
}
```

file1.cpp
```cpp
inlclude "file1.h"

// 正式定义时额外加一个extern
extern int jeason::a=123;
```

别的文件使用：
```cpp
inlclude "file1.h"

extern const int jeason::a;
```

所以，**对于直接定义在文件中的变量，不管是不是const类型的变量，定义时是都加上extern就完事儿了。**对于作为类成员变量的const变量，按类的规则访问。

### 2.4.1 const类型的引用

对const类型变量的引用，简称为**常量引用**。


#### 2.4.1.1 常量引用可使用类型转化进行初始化

```cpp
const int bufferSize = 512;

// 这样定义的话，会编译报错：
// 将 "int &" 类型的引用绑定到 "const int" 类型的初始值设定项时，限定符被丢弃
int &rBufferSize = bufferSize;

// 必须要用下面的方式定义  对常量的引用
const int &rBufferSize2=bufferSize;
```

可见，**引用的类型必须与所引用的对象的类型保持一致**，但是，有两个例外，此处出现第一个例外：

常量引用初始化时，可以使用非常量，并且能自动转化为相应类型，比如：

```cpp
int i = 42;
double d = 3.14;
const int ci = 66;

const int &ri = i;
const double &ri2 = d;//double自动截断为int
const int &ri3 = 32;
const int &ri4 = ri3 * d;

//上面的全是合法的，仅这一句不合法，因为：
// 只有对常量的引用进行初始化时，才可以类型不完全一致
int &i4 = d;
```

为什么 const double &ri2 = d;//double自动截断为int  这句都可以，这个 int &i4 = d;不行呢？

```cpp
double d=3.14;

const int &rI=d;
// 上面这一句编译器自动进行了如下转换：

const int temp=d;
const int &rI=temp;//常量引用自动绑定到了一个临时量上
```

可见，常量引用其实是绑定到了一个临时量上，假设一个非常量引用也进行相似的转换，则是：

```cpp
double d=3.14;

int &rI=d;//实际上是不合法的
// 上面这一句编译器自动进行了如下转换：

int temp=d;
int &rI=temp;//非常量引用自动绑定到了一个临时量上
```

那么，经过类型转换后，rI绑定到了一个临时量，而不是绑定到了d上，而且 rI不仅仅可读，甚至可以写，**通过rI写一个临时量显然没有意义**，因此，对于非常量的引用直接不允许利用类型转初始化是最理想的。

因此，**恰恰因为常量引用只读的特殊性，允许常量引用的初始值类型相对宽松：能类型转换即可**。


#### 2.4.1.2 常量引用可能引用一个非const对象

```cpp
double value = 1.12;
const int &crValue = value;//实际上引用的一个临时量，始终指向那个临时量

cout << crValue << endl;//输出1
value = value + 1;
cout << crValue << endl;//输出1


int iIvalue = 1;
const int &criIvalue = iIvalue;//没有临时量，直接是原对象

cout << criIvalue << endl;//输出1
iIvalue = iIvalue + 1;//原对象变了，引用跟着一起变
cout << criIvalue << endl;//输出2
```

仅能对常量引用进行读操作，如果常量引用绑定了一个非const对象，原对象可以读写，但是这个常量引用还是只能读。


### 2.4.2 指针和const
类似与常量引用，指向常量的指针，可以简称为**常量指针**。

```cpp
double value = 1.12;
const int *cpValue = &value; //编译报错，类型不匹配，无法初始化

int iIvalue = 1;
const int *cpiIvalue = &iIvalue; //直接是原对象

cout << *cpiIvalue << endl; //输出1
iIvalue = iIvalue + 1;      //原对象变了，指针所指向的跟着一起变
cout << *cpiIvalue << endl; //输出2
```

**和常量引用一样，常量指针也没有规定指向的变量必须是常量**，并且，常量指针不接受类型转换的值的地址。

#### 2.4.2.1 const指针
常指针：指针本质是变量，它也有值，**不允许修改指向的对象**的指针成为常指针。常指针和引用什么相似，只又一次机会绑定具体的对象。

```cpp
int value = 123;

// 和普通常量、引用一样，声明就必须进行初始化
int *const pvalue = &value;

//指向常量（此处并不是真的常量）的常指针
const int *const cpValue = &value;
const int *const cpValue2 = cpValue;
```

### 2.4.3 顶层const和底层const
正如前面描述，由于指针的特殊性，指针指向一个const类型的对象和指针本身是不是const是两个独立的问题，为了简化描述这两个问题的描述，使用**顶层const**和**底层const**进行区分。

顶层const：指针本身就是const，也就是 int *const pInteger=&i；更一般的，某个变量本身是const（比如，const int i=1），那就是顶层const。

底层const：指针指向一个的const值，也就是 const int *pInteger=&i；

对指针进行拷贝操作时，必须满足以下**其中一个**条件：

1. 源和目标具有相同的底层const
2. 源到目标 的数据类型可以自动完成转换，比如，非常量可以转换为常量（此处的非常量转换为常量，只是无法通过常量指针修改值，但是修改原值会影响常量指针就解引用得到的值，比如：int a=1；const int *p=&a；这种情况），反之不行。


### 2.4.4 constexpr类型和常量表达式
常量表达式（const express），是指**值不会改变**，并且在**编译时就能得到结果**的表达式，比如预编译时进行值替换的NULL。

一个对象/表达式是不是常量表达式，由它的数据类型和初始值共同决定：

```cpp
const int max_files=20; //不会改变+编译期确定

const int limit=max_files+1;//不会改变+编译期确定

int buffSize=123;
//编译期确定，但是，其类型是普通的int，不是const int，能被改变


const int sz=get_size();//假设get_size只有运行时才能有值，sz则无法在编译期确定值，不是常量表达式
```

**int和const int算不同的类型！！！！！**

在复杂的系统中，有时候表达式十分复杂，为了让编译器检查某个变量是不是常量表达式，可以类型声明为constexpr 类型：

```cpp
constexpr int mf=20;

constexpr int limit = mf+1;

constexpr int size=size();//根据size能否在编译期就进行计算，由编译器检查size是不是真的常量表达式
```
所以，constexpr在定义（定义=声明+初始化）变量的行为中相当于做了两件事：

1. 类型声明为const
2. 检查初始化用的表达式能否在编译器得到一个确定的值

所以，**如果程序员认为某个变量是常量表达式就把它声明为constexpr类型**。

#### 2.4.4.1 字面值类型

常量表达式需要在编译时就得出确定的值，因此，在编译期就能参与到常量表达式的变量类型其实非常有限（因为这个时期都还没有类、结构体），这种能在编译期就能参与常量表达式计算的类型，叫**字面值类型**。目前为止，介绍过得字面值类型只有 算数类型、引用和指针。

#### 2.4.4.2 指针和constexpr

constexpr修饰指针时，只能起到顶层const的作用：

```cpp
const int *p=nullptr;//常量指针，指向常量的指针，此处是底层const

constexpr int *q=nullptr;//指针常量，q永远指向nullptr的地址，constexpr只有顶层const作用

constexpr const int *m=nullptr;//constexpr是顶层const，const是底层const

//i定义在函数体之外的情况下，比如直接定义在文件中
constexpr const int *pValue=&i;
```


## 2.5 处理类型
随着程序越来越复杂，且cpp目前为止还没有正式的package的概念，只能用namespace来模拟package，从而避免变量名的重名、相互污染。

而且勤于写namespace的人比较少，所以类名现在一般都很长 或者 有些沙雕用一些莫名其妙的缩写，所以，目前的cpp，变量名使用起来困难重重。

基于以上原因，我们需要对类型进行一些处理。

### 2.5.1 类型别名
类型别名（type alias）是一个名字，它是某种类型的同义词。但是，要是原始的名字过于底层实现，倒是可以起个别名有助于业务理解。

有两种方法定义别名：

1. 传统的typedef关键字，比如，typedef double sb;
2. 使用新标准中的**别名声明**，比如，using sb=double;

```cpp
{
    const char *title = "start tyedef and using";
    cout << title << endl;

    typedef double SJD; //双精度
    SJD sjd = 123;
    double d = sjd;

    using SJD_2 = double;
    SJD_2 sjd_2 = 666;

    cout << sjd + sjd_2 << endl;
}
```

#### 2.5.1.1 指针、常量、类型别名
如果某个类型的指代的是复合类型或者常量，那这个别名类型用到声明/定义中常常让人懵逼，甚至自己也懵逼。比如：

```
typedef char *pChar;
//pChar是一个char指针类型
//一开始我写的是 typedef char* pChar;
//经过自动格式化。*被和pChar放在一起……

char a = '1';

pChar pa = &a; //a是指向a的指针

cout << pa << endl;

const pChar cstr = 0;//cstr是指向char型的常量指针，是顶层const

const pChar *ps;//ps是一个指针，它指向  （指向char型常量指针）

```

现在解析一下的 const pChar到底是啥复合类型：按照从右往左看的原则，第一个类型符是pChar，这表明被定义的是一个表明是个指向char的指针，然后，再用const修饰修饰这个指针，限定这个指针不能变，因此，const pChar实际上是表示顶层const。**一定要从右向左看类型。**很多人会尝试文本替换来理解：

```cpp
typedef char *pChar;
const pChar cstr = 0;//cstr是指向char型常量指针

//经过文本替换：
const char *cstr=0;
//经过错误的文本替换方式理解，cstr被错误的理解为 指向一个 char型常量的  指针
//因为const char是个复合类型
//上面的定义实际上是：
(const char) *cstr=0;

//很明显了，被误解为一个指向char型常量的指针

```

### 2.5.2 auto类型说明符
C++11引入该说明符，让编译器自动推断类型。比如：

```cpp
auto i=0,*p=&i;//正确，i首先被推断为int，p再推断为int指针
auto sz=0,pi=3.14;//错误！！sz首先被推断为的int，但是，再用这个类型取定义pi时类型不符合，有点类似于初始化列表也不允许精度丢失
int j = 1, k = 1.23;//自己明确类型时就允许由浮点数截断为int
```

#### 2.5.2.1 复合类型、常量和auto
编译器推断出来的auto类型有时候和初始值的类型并不完全一样，编译器会适当地改变结果类型使其更符合初始化规则。适当改变类型的情况有如下几种：

1. 忽略引用类型，使用引用绑定的对象的类型作为推断类型：

```cpp
auto i=123, &r=i;
auto a=r;//a被推断为int，而不是int &
```

2. auto一般会忽略顶层const，保留底层const，比如：

```cpp
int i=1;
const int ci=1,&cr=ci;

auto b=ci;//b是int，顶层const 被忽略
auto c=cr;//c是int，顶层const 被忽略

auto d=&i;//推断为int*
auto e=&ci;//推断为 const int *e=&ci，保留了底层const
```

**注意，声明/定义时，符号&和*只属于某个声明符/变量名，而非基本数据类型的一部分。**

#### 2.5.3 decltype类型指示符
有些情况下，想根据表达式的返回值类型，申请一个同类型的变量，但是又不想直接用表达式的值作为初始值，就使用decltype关键字。这是C++11中新增的关键字……我擦，这么傻逼提案都能过……不增加Module特性，加这种傻逼的关键字……

```cpp
int a=2333;
decltype(a)  value;

int &b=a;
decltype(b) c;//报错，c被声明为int &，但是又未初始化……好傻逼的关键字……
```

**注意点，引用的类型会直接被检测为引用类型，和auto不太一样，auto根据所绑定的原始对象类型进行推断。**

## 2.6 自定义数据结构

### 2.6.1 定义SaleData类

```cpp
#ifndef SaleData_h
#define SaleData_h
namespace bean
{
    namespace jeason
    {
        struct SaleData
        {
            std::string bookNo;
            unsigned soldCount = 0;
            double price = 0.0;
        }; //此处必须以分号结尾，因为之后可以继续定义其东西
    } // namespace jeason
} // namespace bean

#endif
```

### 2.6.2 使用SaleData类
略

### 2.6.3 编写自己的头文件
尽管可以在函数中定义一个类，但是这样定义的类仅能在当前函数中可见，所以类一般都是直接声明在头文件中，并在相关的文件中对声明进行实现（也就是定义，cpp支持多次声明+单次定义）。

#### 2.6.3.1 预处理器概述
确保头文件被多次仍能安全工作的常用技术就是**预处理器**，cpp额该项技术继承自c。

预处理器是在编译之前执行的一段程序，可以部分的改变我们所写的程序，比如：#include<iostream>，预处理器看到该命令后就会用iostream头文件的内容代替#include<iosteram>。

CPP为了避免头文件的循环、重复包含，用到了一项功能就是**头文件保护符**，本质上就是通过判断某变量有没有被定义，被定义就说明该头文件已经包含过了，判断所依据的某变量是**预处理变量**。

预处理变量有两种状态：已定义和未定义。

#define，将一个名字设为预处理变量

#ifdef，当且仅当变量名已经被定义时，结果为真

#ifndef，当且仅当变量名未定义时，结果为真

#endif，#ifdef和#ifndef为真时就会执行后续的操作，直到遇到#endif为止

比如：

```cpp
#ifndef SaleData_h
#define SaleData_h

#include <string>

namespace bean
{
    namespace jeason
    {
        struct SaleData
        {
            std::string bookNo;
            unsigned soldCount = 0;
            double price = 0.0;
        }; //此处必须以分号结尾，因为之后可以继续定义其东西

        void printSaleDate(SaleData &saleData);
        void printSaleDate(SaleData &&saleData);

        SaleData create();

    } // namespace jeason
} // namespace bean

#endif
```

预处理器通过检查预处理变量是否被定义，来判断当前头文件是不是已经被包含了一次，从而避免同一头文件的多次包含。

**头文件保护符一定要习惯性加上。**

# 3 字符串、向量和数组
第三章开始介绍string和vector，可以说是说是两种最重要的标准库数据类型，这两个数据类型与内置的数组类型（此处值C风格的数组，而不是std::array数组）相比，更加安全和灵活。

在介绍标准库之前先学习访问标注库的方法，命名空间。

## 3.1 命名空间的using声明
通过作用域操作符（::)访问域中声明/定义的类、变量、函数、结构体等。

头文件中不应该包含using声明，因为在预编译阶段，头文件会被原封不动的拷贝到include出，如果别的头文件也恰好使用了相同的using声明，那在预编译阶段完成后，就会存在很多重复的using声明，using声明本身重复，没啥问题，但是！！！如果头文件引入的命名空间声明里恰好也有string,而引用头文件的人的也偷懒使用了using namespace std;，然后就直接使用string，那么编译器就无法知道string究竟是std还是头文件中引入的命名空间定义的，编译阶段就会报错。

**综上，头文件中不应该使用using namespace的声明，以避免对调用者变量名的污染。**

## 3.2 标准库类型string
标注库类型string表示可变长的字符序列。和golang一样，CPP的标准库对实现也有较高的要求。

### 3.2.1 定义和初始化string对象

```cpp

string s1;

//以下三种定义初始化完全等价
string s2 = s1;//并不是调用赋值符号，而是实际上使用下一行的拷贝构造函数
string s3{s1};
string s4(s1);

string s5("123");
string s6{"123"};
string s7(10, 'a');
string s8{10, 'a'};

```

注意直接初始化和拷贝初始化的区别！！


### 3.2.2 string对象上的操作
一个类除了规定初始化对象的方式之外，还要定义对象上所能执行的操作。其中，能执行的操作包括函数名和操作符（典型的<<、>>、=、+），在java中不能重载操作符。


#### 3.2.2.1 读写string对象

```cpp
string s9;
cin >> s9;//调用了string的>>操作符
cout << s9;//调用了string的<<操作符
```

其中，string 的>>操作符定义中，其会忽略开头的空格/换行，直到遇到第一个空格结束，比如，输入"    helloWorld    "时，只会输出"helloWorld"。


#### 3.2.2.2 读取未知数量string对象

```cpp
string s10;
//当输入流无效时，不进入循环内
//输入结束或者非法输入时，>>的返回值会无效
while (cin >> s10) {
    cout << s10 << endl;
}
```
#### 3.2.2.3 使用getLIne读取一整行

之前从cin读取内容到string，>>都会忽略开头的空白，但是有时候我们有读取所有内容不丢失空格的需求，这时候就该使用getLine()函数了，**这个函数和其他操作符一样，直接定义在头文件中，而不是作为类的方法，其实算是std域内的静态方法了，因此调用的时候不依赖类实例名和类指针名**。但是，为什么我映像里记得有的类重载操作符时，用到了this指针，运算符被定义成public的成员函数？？？？？？？？？？？？


```cpp
string s11;
//每次读取一整行，遇到换行符时停止读取，并将读到的存入string，
//存进string的部分不包含最后读到的换行
while (getline(cin, s11)) {
    cout << s11;
}
```
一定要主要getline得到的字符串并不包含换行符！！！比如，当这一行只有一个换行符，那得到的string其实是长度为零的字符串，即为空字符串。

#### 3.2.2.4 empty和size操作
```cpp
string s12;
while (getline(cin, s12)) {
    if (!s12.empty()) {
        cout << s12.size();
    } else {
        cout << s12.size();
    }
}
```

string::sizte()的源码如下：

```cpp
typedef typename _Alloc_traits::size_type		size_type;
///  Returns the number of characters in the string, not including any
///  null-termination.
size_type
size() const _GLIBCXX_NOEXCEPT
{ return _M_string_length; }
```

返回值是string::size_type类型，不是size_t。string类和其他大多数标准库类型基本都会定义一些和自己功能搭配的类，这些配套类型目的基本都是：实现平台无关性、方便理解。

从size()的语义上可以推测，string::size_type肯定是无符号整型的且数值足够大。并且，当有符号和无符号一起运算时，有符号会被自动转为无符号，因此，**string::size_type绝对不要和int混合在一起运算！！！**，比如：

```cpp
string s13 = "12345";
int i = -1;
//输出真，因为-1会被转为一个很大的无符号整型
cout << (s13.size() < i) << endl;

```

还是再强调一下，string::size_type绝对不要和int混合在一起运算！！！

#### 3.2.2.4 两个string对象相加

会返回一个全新的对象！源码如下：

```cpp
template<typename _CharT, typename _Traits, typename _Alloc>
basic_string<_CharT, _Traits, _Alloc>
operator+(const basic_string<_CharT, _Traits, _Alloc>& __lhs,
        const basic_string<_CharT, _Traits, _Alloc>& __rhs)
{
    basic_string<_CharT, _Traits, _Alloc> __str(__lhs);
    __str.append(__rhs);
    return __str;
}
```

可见，构造了一个全新的对象，并进行了append操作。

当字符串字面值和string对象相加时，STL针对 字面值+string和 string+字面值  两种情况进行了重载：

```cpp
# const char * + string对象
  template<typename _CharT, typename _Traits, typename _Alloc>
    inline basic_string<_CharT, _Traits, _Alloc>
    operator+(const basic_string<_CharT, _Traits, _Alloc>& __lhs,
	      const _CharT* __rhs)
    {
      basic_string<_CharT, _Traits, _Alloc> __str(__lhs);
      __str.append(__rhs);
      return __str;
    }

# string对象+const char *
  template<typename _CharT, typename _Traits, typename _Alloc>
    basic_string<_CharT, _Traits, _Alloc>
    operator+(const _CharT* __lhs,
	      const basic_string<_CharT, _Traits, _Alloc>& __rhs)
    {
      __glibcxx_requires_string(__lhs);
      typedef basic_string<_CharT, _Traits, _Alloc> __string_type;
      typedef typename __string_type::size_type	  __size_type;
      const __size_type __len = _Traits::length(__lhs);
      __string_type __str;
      __str.reserve(__len + __rhs.size());
      __str.append(__lhs, __len);
      __str.append(__rhs);
      return __str;
    }

```

string并没有对 字面值+字面值 的形式进重载+运算符，因此，要想使用string的+运算符，则必须保证+的一侧有一个string类型。并且，cpp标准库中并没有针对 字面值 相加的运算符重载。

### 3.2.3 处理string对象中的字符
cpp将c语言ctype.h头文件中的一系列函数整理为cpp头文件的形式，并且声明在std命名空间，cpp相应的头文件为cctype。

#### 3.2.3.1 使用范围for处理字符串的每个字符

cpp11中引入范围for语句，跟java中的范围for一毛一样，如：

```cpp
#include <iostream>
#include <cctype>


int main() {
    {
        std::string s1 = "123";
        std::isalpha(s1[0]);

        for (auto each:s1) {
            //string对象中，每个子序列是char型
            //每次迭代，如果有下一个char，则下一个char被拷贝给each变量
            each;
            std::cout << each << std::endl;
        }


        std::string s2 = "adhakdhkada!!!!k aoeoqeaw aeuoajdlad";
        //统计标点符号的个数
        decltype(s2.size()) punctNum = 0;
        for (auto each:s2) {
            //使用cctype中的方法
            if (ispunct(each)) {
                ++punctNum;
            }
        }


        //从上面的代码和注释中看出，每次循环迭代，each的值始终是值拷贝，
        // 那如何通过范围for直接改变string中的字符？？？
        for (auto &eachRef:s2) {
            //通过的C库中的方法将char转为大写
            eachRef = std::toupper(eachRef);
        }
    }

}

```

#### 3.2.3.2 通过下标针对处理部分字符
P84