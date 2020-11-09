# 1 背景

参考文档：

C++ 11完美转发   https://blog.csdn.net/liujiayu2/article/details/49279419

# 2 详解


# 3 举例及实践

make_unique的使用容纳对象的有参构造时，内部实现使用了完美转发
```cpp
#if !defined(BOOST_NO_CXX11_VARIADIC_TEMPLATES)
template<class T, class... Args>
inline typename detail::up_if_object<T>::type
make_unique(Args&&... args)
{
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}
#endif
```
