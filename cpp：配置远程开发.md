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
安装插件，然后添加远程的机器即可，没啥可讲的。

## 3.2 cmake tools
cmake本质上是个跨平台的项目描述，可将cmake项目转换为vs、makefile等流行的项目结构，虽然的微软提供的插件叫cmake-tools，但是这个插件提供的的功能有：
1. 将项目转换为makefile结构，可选unix makefile还是cygwin makefile
2. 对生成的makefile进行make，make会根据makefile描述的编译行为进行编译、链接
3. 可以提供对链接生成的文件的一键运行的按钮，包括调试运行和直接运行

### 3.2.1 配置cmake tool
安装完之后，会在左下角从左往右依次增加了:
1. cmake编译模式（cmake build variant）

点击之后就能切换编译模式，默认自带4种默认配置

2. active kit（使用的工具链）

也就是C和C++的编译器，一般都能直接在系统环境变量里找到相应的可执行文件

3. Build按钮，后面还带个[all]这样的方括号

[all]时，点击build按钮会编译cmake里描述的所有的target。点击[all]可以切换target。如果cmake比较新，cmake tools会使用serverApi，切换target会自动列出所有的target，而旧的cmake我们只能手动输入target名字进行指定。


4. 一键调试和一键运行按钮

对于旧的cmake，不支持这两个按钮，只有新的cmake才支持，新的cmake支持一键调试/运行，当前的target。

一键调试的功能比较鸡肋，没有变量列表，没有断点列表……还是得鼠标悬浮到变量上才能看到对象实例具体的内容。所以，调试还是要用vscode自带的调试，自己配置才行，下文有具体的流程。

一键运行，倒是没啥问题，用起来还可以。

5. 修改/确认 cmake tools的配置。

注意一下几个cmake tool的配置：

```json
# 注意编译的生成目录
"cmake.buildDirectory": "${workspaceFolder}/build"

# 默认使用auto即可，新的cmake会自动使用serverApi，旧的会使用legacy
"cmake.cmakeCommunicationMode": "automatic"

# 建议留空，默认好像是unix makefile，反正别再Linux环境中使用Windows中的，比如Cygwin makefile
"cmake.generator": null
```

6. 利用vscode自带工具进行debug
左上角“运行”——“添加配置”，会弹出来launch.json的文件编辑，对于各个参数的意义，自己鼠标悬停上去查看含义。
```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "(gdb) 启动",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/build/exe01",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ]
        }
    ]
}
```
稍微做一些修改，注意其中的 "program": "${workspaceFolder}/build/exe01"  ，这个要设置成可执行文件的位置，也就是build之后生成的文件的位置。保存文件，并通过“运行”————“启动调试”调试一次，"(gdb) 启动"这个任务名称就添加到vscode的状态栏上了。

本质上，配置的这个launch任务，其实就是描述了  gdb xxxxxxx  这样一句命令行，然后vscode根据这个任务的类型是debug类型的，会自动帮我们把内存中的变量、堆栈、断点列表在左侧显示出来，比cmake tool自带的一键调试稍微好用一点。


## 3.3 C++ intelliense
安装插件，没啥可讲的。


## 3.4 典型的工作流程示例


典型的工作流程是：

1. 在CmakeLists中增加一个target
2. 修改build的target
3. 通过一键调试/运行 生成的可执行文件，或者 通过launch.json添加到任务栏中的调试方式
