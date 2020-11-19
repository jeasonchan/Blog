# 1 前言

（20201119更新，文章已过时，建议直接看  https://github.com/jeasonchan/Blog/blob/master/cmake%EF%BC%9Atarget%E9%A3%8E%E6%A0%BC%E7%9A%84%E6%9E%84%E5%BB%BA%E8%A7%84%E5%88%99.md  ）

参考文章：

简明教程简洁易懂，从简单到复杂    https://blog.csdn.net/whahu1989/article/details/82078563

导入第三方静态库（target描述的风格）     https://zhuanlan.zhihu.com/p/108502992?utm_source=com.alpha.pinbox&utm_medium=social&utm_oi=37750522773504

clion的jetbrain cmake教程    https://www.jetbrains.com/help/clion/2020.2/quick-cmake-tutorial.html#profiles


## 1.1 简介

CMake（Cross-Make）是一个跨平台的构建工具，CMake本身并不参与工程的构建和编译，**CMake使用简单的语法描述项目构建的细节，根据用户所需，生成makefile或者IDE的project**。通常存放CMake工程描述细节的文件名是CMakeLists.txt。

CMake的重点就是学习CMake的构建脚本（CMakeLists.txt）及CMake描述工程的逻辑。

对于Cmake的逻辑，总结如下：

CMAKE根据变量（包括CMake预定义的变量和用户定义的变量，比如CMAKE_CXX_STANDARD）来确定当前的构建环境和构建规则（构建环境包括依赖的   头文件搜索目录位置、库搜索目录、要链接的库文件），**修改构建环境和构建规则本质上就是在修改这些变量**。

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

## 2.4 多目录多cmakelists.txt：add_subdirectory

现在有很多很多正规cmake项目都是如下的目录结构：

|    |    |
| ---| ---|
| bin |      |
| build |      |
| include      |      |
|  ----------    |   testFunc1.h|
|  ----------    |   testFunc.h|
| src     |      |
|  ----------    |   testFunc1.c|
|  ----------    |   testFunc.c|

在顶层目录下的新建一个cmake文件：

```bash
cmake_minimum_required (VERSION 2.8)

project (demo)

# 向当前工程添加存放源文件的子目录，
add_subdirectory (src)
```

这里指定src目录下存放了源文件，当执行cmake时，就会进入src目录下去找src目录下的CMakeLists.txt，所以在src目录下也应该建立一个CMakeLists.txt，内容如下：

```bash
aux_source_directory (. SRC_LIST)

include_directories (../include)

# 目标是输出可执行文件
add_executable (main ${SRC_LIST})

# 设置输出的可执行文件的位置
# EXECUTABLE_OUTPUT_PATH  这个变量是cmake自带变量
set (EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
```

最终整个工程的目录结构如下的：

|    |    |
| ---| ---|
|  cmakelist.txt |   |
| bin |      |
| build |      |
| include      |      |
|  ----------    |   testFunc1.h|
|  ----------    |   testFunc.h|
| src     |      |
|  ----------    |   cmakelist.txt|
|  ----------    |   testFunc1.c|
|  ----------    |   testFunc.c|

编译执行时，cd到build目录下，执行cmake .. && make  即可。



# 3 两种风格导入第三方库（OpenCV）

## 3.1 面向过程风格

```bash
# 指定需要CMake的最低版本
cmake_minimum_required(VERSION 3.14)

# 设置工程名
project(test)

# 设置变量CMAKE_CXX_STANDARD的值为17
set(CMAKE_CXX_STANDARD 17)

# 设置opencv的根目录变量、头文件目录变量、链接库位置变量、链接库名称变量
set(opencv_root /home/ding/develop/lib/opencv/opencv-3.4.9)
set(opencv_include ${opencv_root}/include)
set(opencv_lib_dir ${opencv_root}/lib)
set(opencv_libs libopencv_world.so.3.4.9)

# 设置头文件目录为opencv_include的值
include_directories(${opencv_include})

# 设置链接库目录为opencv_lib_dir的值
link_directories(${opencv_lib_dir})

# 设置要链接文件的名字为opencv_libs的值
link_libraries(${opencv_libs})

# 添加构建可执行文件的目标demo，目标包含main.cpp源文件
add_executable(demo main.cpp)
```

在这个例子中，想要修改c++的版本，只需通过set语句修改CMAKE_CXXSTANDARD这个变量的值，那么cmake在生成project的时候，将会根据这个变量的值来设置c++的语言版本。具体到编译参数，如gcc，就是加一个-std=c++17参数。再比如想要使用pthread库，将CMAKE_EXE_LINKER_FLAGS变量附加一个-lpthread即可，即：set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} -lpthread)

## 3.2 target风格

上面设置OpenCV依赖时使用了include_directories、link_directories、link_libraries三个函数添加头、库，和cmake的find_package语句的原理是一致的，只不过find_package使用的OpenCV_DIR变量和库本身提供的xxxx.cmake文件，转换为find_package如下：

```bash
# 指定需要CMake的最低版本x
cmake_minimum_required(VERSION 3.14)
# 设置工程名
project(test)
# 设置变量CMAKE_CXX_STANDARD的值为17
set(CMAKE_CXX_STANDARD 17)

# 设置OpenCV_DIR变量
set(OpenCV_DIR /home/ding/develop/lib/opencv/opencv-3.4.9/share/OpenCV)

# 查找OpenCV，其实就是使用 OpenCV_DIR 目录下xxxx.cmake文件
find_package(OpenCV REQUIRED)

# 添加可执行构建目标demo
add_executable(demo main.cpp)

# 为demo目标设置链接库
target_link_libraries(demo ${OpenCV_LIBS})

# 设置包含目录
target_include_directories(demo ${OpenCV_INCLUDE_DIR} PRIVATE)
```

对于OpenCV，用CMake构建安装后，会生成相应的CMake支持文件，3.x版本cmake支持文件在<install_path>/share/OpenCV，4.x版本在<install_path>/lib/cmake/opencv4目录下，所以上面的cmakeLists.txt中设置OpenCV_DIR要注意根据不同的版本设置不同的路径。

### 3.2.1 find_package的原理

find_package在如下目录中搜索\<package_name\>Config.cmake文件或Find\<package_name\>.cmake文件：

* \<package\>_DIR
* CMAKE_PREFIX_PATH
* CMAKE_FRAMEWORK_PATH
* CMAKE_APPBUNDLE_PATH
* PATH

如果搜索成功，将执行\<package\>Config.cmake或Find\<package\>.cmake文件，其中\<package\>Config.cmake文件用于config模式，Find\<package\>.cmake用于moudle模式，cmake优先使用moudle模式。执行完后，将**自动添加**如下变量：

* \<package\>_FOUND
* \<package\>_INCLUDE_DIRS or <package>_INCLUDES
* \<package\>_LIBRARIES or <package>_LIBRARY or <package>_LIBS
* \<package\>_DEFINITIONS

所以，上面cmakelsits最后target命令使用的几个变量，都是找到xxxx.cmake后自动添加的：

```bash
# 为demo目标设置链接库
target_link_libraries(demo ${OpenCV_LIBS})

# 设置包含目录
target_include_directories(demo ${OpenCV_INCLUDE_DIR} PRIVATE)
```

以target开头cmake函数是另一套api，在新的文章中讲述。

## 3.3 find_package综合：同时构建QT5和OpenCV

```bash
cmake_minimum_required(VERSION 3.5)

project(one LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

# 打开全局moc
set(CMAKE_AUTOMOC ON)
# 打开全局uic
set(CMAKE_AUTOUIC ON)
# 打开全局rcc
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置OpenCV的CMake支持文件目录
set(OpenCV_DIR /home/ding/develop/lib/opencv/opencv-3.4.9/share/OpenCV)
# 设置Qt5的CMake支持文件目录
set(Qt5_DIR /home/ding/develop/sdk/qt/Qt5.12.7/5.12.7/gcc_64/lib/cmake/Qt5)

find_package(Qt5 COMPONENTS Core Gui Widgets Qml Quick REQUIRED)
find_package(OpenCV REQUIRED)

add_executable(one
    main.cpp
    mainwindow.cpp
    mainwindow.h
    mainwindow.ui)

target_link_libraries(one Qt5::Core Qt5::Gui Qt5::Widgets)
target_link_libraries(one ${OpenCV_LIBS})
```


# 7 总结
