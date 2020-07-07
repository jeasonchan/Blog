# 1 前言

参考文章：

简明教程简洁易懂，从简单到复杂    https://blog.csdn.net/whahu1989/article/details/82078563

导入第三方静态库（target描述的风格）     https://zhuanlan.zhihu.com/p/108502992?utm_source=com.alpha.pinbox&utm_medium=social&utm_oi=37750522773504

如何评价 CMake？ - 邱昊宇的回答 - 知乎（基于 target 描述的模型）           https://www.zhihu.com/question/276415476/answer/537782595

clion的jetbrain cmake教程    https://www.jetbrains.com/help/clion/2020.2/quick-cmake-tutorial.html#profiles

# 2 


# 7 总结
1. cmake官方并不推荐aux_source_dir、file这类的命令进行代码文件的添加。因为，使用了这样笼统的命令之后，dev向工程中添加源文件之后，cmakelists.txt很可能完全不需要变化，cmake也就无法感知到工程发生了变化，就会生成
