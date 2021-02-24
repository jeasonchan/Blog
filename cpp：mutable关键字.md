# 0 背景
参考文章：

cpp手册   https://en.cppreference.com/w/cpp/language/cv   ，手册里给出的例子非常好


# 1 语义
const修饰一个对象时，该对象就只能被调用cosnt类型的函数，或者，自己强转为普通对象，也能任意调用非const成员函数了。但是！有时候真的需要

permits modification of the class member declared mutable even if the containing object is declared const.

Mutable is used to specify that the member **does not affect the externally visible state of the class** (as often used for mutexes, memo caches, lazy evaluation, and access instrumentation).

