# 1 前言
之前用cpp的时候，加、减、乘、除、赋值符号什么的，都只是简单的看作操作符，也就重载过复赋值符号和加减号什么的，现在回头重新学习，发现：

cpp中的加减乘除赋值符号什么的都只是"普通函数"，只不过函数名、入参、调用方式有点特殊罢了。

以cout和endl为例，以前以为是固定搭配用法，也简单以为endl是个代表换行符的字符串变量，今天看了ostream的<<操作符重载和endl，才发现并没有那么简单。

# 2 解析
先看最常用的cout和endl的用法：

```cpp
cout<<"初始化为："<<number1<<" "<<number2<<endl;
```
显然cout是一个std中的一个实例对象，类型是ostream类的值类型（不是指针也不是引用），那<<符号便是ostream类型的重载，endl其实是std里面的一个函数名（其实，**cpp里面的函数名是个函数指针**）。

<<符号的重载和endl的定义如下：

```cpp

//针对字符串常量的重载
  template<class _Traits>
    inline basic_ostream<char, _Traits>&
    operator << (basic_ostream<char, _Traits>& __out, const char* __s)
    {
      if (!__s)
	__out.setstate(ios_base::badbit);
      else
	__ostream_insert(__out, __s,
			 static_cast<streamsize>(_Traits::length(__s)));
      return __out;
    }

//  <<   针对int的重载太长了，暂时不看了

//  <<   另一个重载，用于处理endl作为入参
    __ostream_type&
    operator<<(__ostream_type& (*__pf)(__ostream_type&))
    {
// _GLIBCXX_RESOLVE_LIB_DEFECTS
// DR 60. What is a formatted input function?
// The inserters for manipulators are *not* formatted output functions.
return __pf(*this);
    }

  template<typename _CharT, typename _Traits>
    inline basic_ostream<_CharT, _Traits>&
    endl(basic_ostream<_CharT, _Traits>& __os)
    { return flush(__os.put(__os.widen('\n')))


//另一个版本的<<重载
ostream& ostream::operator << ( ostream& (*op) (ostream&))
{
// 通过函数指针进行函数调用，解引用+入参
return (*op) (*this);
}

//另一个版本的endl
std::ostream& std::endl (std::ostream& strm)
{
    // write newline
    strm.put('\n');
    // flush the output buffer
    strm.flush();
    // return strm to allow chaining
    return strm;
}

```

可以看出，针对endl的运算符重载函数中的**形参为一个函数指针，其指向一个输入输出均为ostream类引用的函数**。而endl正是这样一个函数。所以我们在运行:

```cpp
cout<<endl;
```

语句时，endl是一个<<这个函数的入参，类型是函数指针。然后会执行：

```cpp
//通过函数指针对函数进行调用
return (*endl) (*this);
```

本质上就是执行了endl函数。endl函数输出一个换行符，并刷新输出缓冲区。

所以对cout调用<<操作符，并将endl作为入参，完全等价于：

```cpp
cout<<endl;
//等价于
endl(cout);
```

