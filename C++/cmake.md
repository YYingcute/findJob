---
layout: post
title: "cmake"
subtitle: 'cmake学习，cpp必备'
author: "zhutao"
header-style: text
tags:
  - cpp
---

cmake官网：https://cmake.org/cmake/help/v2.8.10/cmake.html

参考博客：https://mirkokiefer.com/cmake-by-example-f95eb47d45b1

1. 写CMakeLists.txt

   cmake内置命令不区分大小写，也就是说`add_executable`和`ADD_EXECUTABLE`作用是一样的

2. 创建build文件夹

3. cmake ..

   cmake之后会产生build脚本，也就是Makefile

4. make

```shell
cmake_minimum_required(VERSION 2.8)
project(app_project)    # project name, not so important

include_directories("path/to/.h")
link_directories("path/to/.so and .a")

# ========================================================
# 定义可执行文件的源文件和依赖
# 告诉cmake可执行文件的生成路径
# ========================================================
add_executable(myapp main.cpp)
target_link_libraries(myapp test)
install(TARGETS myapp DESTINATION bin) or SET(EXECUTABLE_OUTPUT_PATH ${CMAKE_SOURCE_DIR})

# =========================================================
# build the library STATIC libtest.a / SHARED libtest.so
# and install .a/.so into lib folder of the install directory
# We also include our public header file into the install step
# and tell cmake to put it into include
# >>> cmake .. -DCMAKE_INSTALL_PREFIX=../_install
# install
# | - bin
# | - lib
# | - include
# =========================================================
install(TARGETS test DESTINATION lib)
install(FILES test.h DESTINATION include)
```

# 链接库

工作中观察到了以下几种不同的so包软链接

- 两个一级软链接

```shell
libcurl.so -> libcurl.so.4.5.0
libcurl.so.4 -> libcurl.so.4.5.0
```

- 一个一级软链接

```shell
libcrypto.so -> libcrypto.so.1.0.0
```

- 一个两级软链接

```shell
libjsoncpp.so -> libjsoncpp.so.0
libjsoncpp.so.0 -> libjsoncpp.so.0.10.6
```

好处如下：

1. 方便动态链接库升级，只需修改软链接，不用修改代码

### cmake链接opencv

```cpp
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)
project(SRHandNet)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
# 寻找opencv
find_package(OpenCV REQUIRED)
#打印调试信息
MESSAGE(STATUS "Project: ${PROJECT_NAME}")
MESSAGE(STATUS "OpenCV library status:")
MESSAGE(STATUS "    version: ${OpenCV_VERSION}")
MESSAGE(STATUS "    libraries: ${OpenCV_LIBS}")
MESSAGE(STATUS "    include path: ${OpenCV_INCLUDE_DIRS}")

add_executable(main main.cpp)
target_link_libraries(main "${OpenCV_LIBS}")
```

### cmake编译动态、静态链接库

```cmake
SET(LIBRARY_OUTPUT_PATH "${CMAKE_CURRENT_LIST_DIR}/lib")	# 设置a，so文件生成文件夹
ADD_LIBRARY(moku_hand_pose_estimate STATIC ${CMAKE_CURRENT_LIST_DIR}/handPose.cpp)
target_link_libraries(moku_hand_pose_estimate ${MNN_DEPS})	# 这句话表示需要依赖其他的so文件
# 其中${MNN_DEPS}表示so文件的名字，比如libMNN_express.so, 如果直接用so文件表示，则写成
target_link_libraries(moku_hand_pose_estimate MNN_express)


# =========================
add_library(<name> [STATIC | SHARED | MODULE]
            [EXCLUDE_FROM_ALL]
            [source1] [source2] [...])
其中<name>表示库文件的名字，该库文件会根据命令里列出的源文件来创建。而STATIC、SHARED和MODULE的作用是指定生成的库文件的类型。STATIC库是目标文件的归档文件，在链接其它目标的时候使用。SHARED库会被动态链接（动态链接库），在运行时会被加载。MODULE库是一种不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数。默认状态下，库文件将会在于源文件目录树的构建目录树的位置被创建，该命令也会在这里被调用。
而语法中的source1 source2分别表示各个源文件。
```

### glob

```cmake
# https://www.cnblogs.com/fnlingnzb-learner/p/7221648.html
file(GLOB variable [RELATIVE path] [globbingexpressions]...)

example:
file(GLOB HAND_SRC ${CMAKE_CURRENT_LIST_DIR}/Moku_Hand_MNN/src/*.h
        ${CMAKE_CURRENT_LIST_DIR}/Moku_Hand_MNN/src/*.cpp
        )

GLOB 会产生一个由所有匹配globbing表达式的文件组成的列表，并将其保存到变量中。Globbing 表达式与正则表达式类似，但更简单。如果指定了RELATIVE 标记，返回的结果将是与指定的路径相对的路径构成的列表。 (通常不推荐使用GLOB命令来从源码树中收集源文件列表。原因是：如果CMakeLists.txt文件没有改变，即便在该源码树中添加或删除文件，产生的构建系统也不会知道何时该要求CMake重新产生构建文件。
```

### 打印信息

在cmake定义一个变量“USER_KEY”，并打印此变量值。status表示这是一般的打印信息，还可以设置为“ERROR”，表示这是一种错误打印信息。

```cmake
SET(USER_KEY, "Hello World")
MESSAGE( STATUS "this var key = ${USER_KEY}.")
```

