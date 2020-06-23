#  1 前言

参考文档：




# 2 vscode
1. 安装vscode
2. 安装微软的C++插件，功能是代码提示、触发调试、代码跳转等


# 3 安装、配置cygwin或者mingw
## 3.1 安装
安装两者的目的是获得其中的g++编译器、gdb调试器，以及c++的公共库的头文件及其链接库。

mingw是gcc在win下的特定实现，尤其是使用了windows的api，如果就是为了写在win上运行的程序直接用mingw即可。



cygwin是linux中一系列命令行工具/可执行文件/公用库的合集，包括但不限于：gcc、g++、gdb、git、bash、ssh等。并且，cwgwin使用了一个界面，让我们选择我们安装那些内容。

只想安装**经典的、纯**C++的开发环境时，只需要安装：
1. gcc-g++，编译器器，对源码进行编译、链接，生成可执行文件
2. gcc-core，STL的链接库，因为我之前没安装这个会抱错缺少shared libraries
3. gdb，调试器，使用gdb.exe 去 启动生成的行的可执行文件，能进行调试

预编译（用到gcc-core里的stl的头文件）、编译（g++）、链接g++）、执行；调试（gdb）；**经典的**C++的开发环境配置这些就够了。

但是，现在有的人用clion、有的vs、有的qt，项目的组织结构会有很大差异，于是出现了make。make通过makefile中的指令去编译、生成可执行文件，到目前已经成为了构建cpp项目的"标准"工具了，和maven相比，感觉就是少了依赖下载功能的maven。因此，还是建议在cygwin中选择安装make。

4. make

##  3.2 配置

只需要将 ./cygwin/bin/  配置进windows的PATH中即可，该路径包含了所有的linux中移植过来的命令，包括上面安装的几个。


# 4 vscode配置task.json、launch.json、c_cpp_properties.json文件

先总体介绍一下三个文件，三个文件都位于项目根目录下的 .vscode/  文件中，三个文件的作用如下：

## 4.1 task.json

为了简化在终端中敲代码，就有了这个配置文件，**相当于是个宏**

默认是没有这个文件的，如何产生这个文件？

右上角"终端"，有几个选项：

* 运行任务，会弹出一个列表，让你选择运行task.json中配置的哪个任务
* 运行生成任务，直接运行 "isDefault": true 的 "kind": "build" 的任务
* 运行活动文件，直接用  ./当前文件名  的方式在终端中执行，很明显，只有在文件头中用 !# /user/bsah  这种方式声明了依赖的启动类型的脚本文件才可以直接运行
* 运行所选文本，用鼠标选中vscode中的一段文本，然后选择该项，会在终端中直接运行所选的文本，省去了文本复制粘贴到终端中的过程

终端————配置任务 经过选择，会自动生成task.json，vscode提供的模板有python、gcc、java、node、maven等等，我的g++的模板如下（也是自动生成的），作用是编译、链接当前的cpp文件，生成一个可执行文件：

```json
{
    // See https://go.microsoft.com/fwlink/?LinkId=733558
    // for the documentation about the tasks.json format
    "version": "2.0.0",
    "tasks": [
        {
            "label": "echo",
            "type": "shell",
            "command": "echo Hello",
            "problemMatcher": [],
            "group": "build"
        },
        {
            "type": "shell",
            "label": "C/C++: g++.exe build active file",
            "command": "D:\\cygwin64\\bin\\g++.exe",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}\\${fileBasenameNoExtension}.exe"
            ],
            "options": {
                "cwd": "${workspaceFolder}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}

```

## 4.2 launch.json

launch.json的作用和task.json一样，都是一键运行一条命令（程序的世界本质上不就是运行一条命令……从计算器的启动，到应用层的程序启动……）。

如何产生这个文件？

"运行"————"添加配置"或者"打开配置"，会让我们选择任务类型，vscode会根据选择任务类型自动生成一个launch.json，里面有很多参数，自己看着这改。

调试模式运行，非调试模式运行  在 launch.json  是两条运行命令，需要自己分别为配置命令。


## 4.3 c_cpp_properties.json

右下角点击一下"win32"或者"linux"，这**只是一个标签，标记当前计算机的类型，并不代表cpp的县城模型是win还是linux的**……然后会弹出在UI中编辑还是在json中编辑，无论选哪一种编辑方式，都会在.vscode 文件夹中生成 c_cpp_properties.json 类型


# 5 配置cmake
尽管make已经通过makefile的方式实现了任意平台下编译结果一致性，但是makefile还是太难写了。于是出了一个cmake，用来生成makefile。但是，cmake实在太香了，**极大减少了makefile的书写**量。用了cmake之后，编译运行的流程变成了：

1. 运行cmake，生成makefile 文件，这一步可以写到task.json中
2. 运行make，根据makefile编译链接，生成可执行文件，这一步可以写到task.json中
3. 直接在终端中./可执行文件  或者 通过launch.json里描述的 运行 或者 调试模式模型

## 5.1 安装、配置cmake
cygwin中可选这个组件，下载安装即可，不用再额外配置环境变量。因为，和g++在同一个路径中。

## 5.2 vscode安装cmake插件

cmake, 用于自己编写是CMakeLists.txt时进行语法提示
