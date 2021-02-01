# 1 背景
办公室机是Windows，开发机和产线都是红帽，日常需要通过ssh的方式在远端进行开发，而代码放在哪里，决定了远程开发的两种模式：
1. 代码在Windows，编译器、gdb、各种库及头文件都在Linux，编译器时要先把代码文件sftp到Linux中，再及逆行编译，visual studio支持这种方式
2. 代码、编译器、gdb、各种库及头文件都在Linux，Windows中只是个UI

接下来分别介绍两种remote方案。

# 2 visual studio

参考文档：

https://www.cnblogs.com/apocelipes/p/10899484.html

https://www.jianshu.com/p/b49d32e05af6


项目较大，甚至是小项目，每次网络传输的时间都不短，体验不好……


# 3 visual studio code

参考文档：

https://www.cnblogs.com/NanoDragon/p/12899430.html

**按照安装的前后顺序**，最主要的就是微软出的三个插件：
1. remote-development
2. cmake tools
3. C++ intelliense

下面分别介绍一下几个插件的用法，尤其是编译、运行、debug相关的cmake tools

## 3.1 remote-development


## 3.2 cmake tools


## 3.3 C++ intelliense