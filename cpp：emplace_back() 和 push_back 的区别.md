# 1 背景

参考文档：https://blog.csdn.net/xiaolewennofollow/article/details/52559364

leetcode刷题时用到了std::vector，想到了新增的emplace_back() 方法，好奇和老的push_back有什么区别，在此记录一下。

同时在实验的过程中，发现emplace_back好像能触发多个入参的构造函数类型转换，并且explict无法限制emplace_push的这一隐式类型转换，只能限制push_back的隐式类型转换，并且，push_back之支持当个入参的隐式转换，而emplace_back支持多入参的类型转换，十分神奇。

