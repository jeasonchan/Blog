# 1 背景

参考文章：

C++中new的用法及显示调用析构函数  https://www.cnblogs.com/fnlingnzb-learner/p/9279039.html


new对象时加括号和不加括号时的差别  https://blog.csdn.net/itworld123/article/details/102731038

C++中使用placement new  https://blog.csdn.net/linuxheik/article/details/80449059


最近的在学习STL，看到分配器时，自定义的constructor和destroy用到了的奇怪的new语法和显式调用析构函数，学习记录一下。（正好看到参考文档的作者因为内存池的总结文章。）

# 2 cpp中new的用法
