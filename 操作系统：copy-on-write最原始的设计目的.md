# 1 背景

看作操作系统导论时谈到copy on write思想，我想这不是为了提高并发读而出现的系统设计思想嘛，和内存管理有啥关系，看了一下发现，copy on write最原始设计目的其实是节省内存资源，而后来发现该思想也能解决并发读和偶尔写入的效率，在此学习记录一下java中copy on write和其原始的设计思想。



参考文档：

再谈 copy-on-write - ZeaTalk的文章 - 知乎     https://zhuanlan.zhihu.com/p/136428913

