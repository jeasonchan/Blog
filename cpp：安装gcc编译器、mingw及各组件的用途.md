# 1 前言

参考文档：

windows下安装MinGW的配置    https://zhuanlan.zhihu.com/p/66197013

要进行cpp的开发，至少需要一下几个东西：

1. 编译器，可执行文件一般叫g++，进行预处理、编译、连接 工作
2. 调试器，可执行文件一般叫gdb。g++ -g 生成带有调试标志的可执行文件后，使用gdb xxxx.exe 启动带有调试标记的可执行文件进行调试
3. STL的头文件、库文件

win下的g++都是gcc在win下的实现，我之前通过cygwin安装了g++、git等，但是，通过cygwin调用可执行文件时，路径都是Cygwin转化的……不是的标准的win下的路径表示……这就导致vscode无法识别cygwin中git……通过cmake添加cygwin安装的的boost时无法识别……于是还是决定使用win下更加主流的mingw，在此处记录一下的mingw各个包的作用。

# 2 还是直接用TDM-GCC吧……

官网：https://jmeubank.github.io/tdm-gcc/download/

其中mingw-w64 based的tdm64-gcc-9.2.0.exe包含的内容有：

64+32-bit MinGW-w64 edition. Includes GCC C/C++, GNU binutils, mingw32-make, GDB (64-bit), the MinGW-w64 runtime libraries and tools, and the windows-default-manifest package.

完美解决。

TDM-GCC就是dev c++内置的工具链。
