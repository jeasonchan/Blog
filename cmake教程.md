# 1 前言

参考文章：

简明教程简洁易懂，从简单到复杂    https://blog.csdn.net/whahu1989/article/details/82078563

导入第三方静态库（target描述的风格）     https://zhuanlan.zhihu.com/p/108502992?utm_source=com.alpha.pinbox&utm_medium=social&utm_oi=37750522773504

如何评价 CMake？ - 邱昊宇的回答 - 知乎（基于 target 描述的模型）           https://www.zhihu.com/question/276415476/answer/537782595

clion的jetbrain cmake教程    https://www.jetbrains.com/help/clion/2020.2/quick-cmake-tutorial.html#profiles


cmakelists中cmake的函数命令大小写不敏感，但是，变量名最好大小写区分。

# 2 从简单到复杂的实践

## 2.1 简单样例

```bash
cmake_minimum_required (VERSION 2.8)

project (demo)

# 表明该cmake的target是生成一个名为main的可执行文件，
# 一个cmake中可定义多个target
add_executable(main main.c)
```

## 2.2 同一目录下多个源文件

```bash
cmake_minimum_required (VERSION 2.8)

project (demo)

add_executable(main main.c testFunc.c)
```

就是手动在add_executable中多添加了几个参数，但是源文件数量上百个的情况下的，就可以使用 aux_source_directory(dir var)  cmake自带的函数。

### 2.2.1 aux_source_directory(dir var_name)函数简介

该命令可以把指定目录下所有的源文件存储在一个变量中，供以后使用，比如：

```bash
# 将src文件夹内的文件都添加仅cppFiles
aux_source_directory(src cppFiles)

# 使用cppFiles内的文件
message(${cppFiles})
add_executable(main ${cppFiles})
```

但是！！！

* aux_source_directory并不能递归子文件夹添加，这点可以通过file命令进行替代，递归将文件添加进变量

```bash
file(GLOB_RECURSE variabl_name [RELATIVE path] [FOLLOW_SYMLINKS] [globbing expressions]...)
```

* cmake官方并不推荐aux_source_directory、file这类的命令进行代码文件的添加。因为，使用了这样笼统的命令之后，dev向工程中添加源文件之后，cmakelists.txt很可能完全不需要变化，cmake也就无法感知到工程发生了变化，就会生成

## 2.3 不同目录下多个源文件
假设现有的文件结构是：

|    |    |     |
| ---| ---| ---|
| cmakelists.txt |      |
| main.c         |      |
| test_func      |      |
|  ----------    |   testFunc.c|
|  ----------    |   testFunc.h|
| test_func1     |      |
|  ----------    |   testFunc1.c|
|  ----------    |   testFunc1.h|


则可以使如下的cmake文件生成makeFile文件：

```bash
cmake_minimum_required (VERSION 2.8)

project (demo)

# 该命令是用来向工程添加多个指定头文件的搜索路径，路径之间用空格分隔。
include_directories (test_func test_func1)

# 要生成两个SRC_LIST变量，
# 因为aux_source_directory会对变量原先的数据进行覆盖
aux_source_directory (test_func SRC_LIST)
aux_source_directory (test_func1 SRC_LIST1)

add_executable (main main.c ${SRC_LIST} ${SRC_LIST1})
```

或者，直接将所有的文件（头文件和源文件）都直接添加add_executable中，clion就是这么做的，新建类后会自动在cmake文件的add_executable中追加新增的头文件、源文件，比如：

```bash
cmake_minimum_required (VERSION 2.8)

project (demo)

add_executable(
    main 
    main.c
    test_func/testFunc.c
    test_func/testFunc.h
    test_func1/testFunc1.c
    test_func1/testFunc1.h
)
```





# 7 总结
