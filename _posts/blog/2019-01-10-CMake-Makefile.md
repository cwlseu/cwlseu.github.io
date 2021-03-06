---
layout: post
title: Make & CMake 进阶
categories: [blog ]
tags: [linux开发]
description: 
---
* content
{:toc} 

## 引言

当下流行的IDE，将源代码生成可执行文件的过程都封装起来，对于开发着来说方便使用。
但是对于初学者来说，蒙蔽了源代码到可执行文件过程。源代码预处理，编译，打包，链接等步骤，
才能形成IDE中的一步到位的可执行文件target。而Makefile是直白的描述一个源代码如何被操作
才能成为target的一种文件格式。而CMake是一种可以通过配置的方式生成Makefile的脚本. 
如果只是简单的开发一个.cpp进行测试，Makefile是首选。
本文中不对Makefile的基本语法进行介绍，要学习基本语法可以参看陈皓老师的Makefile中文教程进行学习。


## Makefile

### `ifeq`和`ifneq`之后要有个空格，否则不识别

```cmake
ifeq ($(UNAME), linux)
     $(info "")
else
      $(warning "")
     $(error $(HAVE_SSHSERVER))
endif 
```

### 定义变量 

`HAVE_THE_VALUE :=`   # 新定义一个变量
`HAVE_THE_VALUE ?=`   # 如果没有定义，则定义一个新变量
`HAVE_THE_VALUE +=`  # 往变量中append数据

这个地方有点像pascal，不要与shell中混淆了


### 变量赋值
**后面一定不要有空格，回车之类的空白符号**，否则可能会将你整疯了的。
就拿caffe中的Makefile.config中

`USE_LEVELDB := 1`

`USE_LEVELDB := 1 `

这两行的区别在于，第二行赋值操作后面有一个空格。在Makefile中通过如下代码进行添加编译需要的宏。

```makefile

ifeq ($(USE_LEVELDB), 1)
  CXX_FLAGS += -DUSE_LEVELDB
endif
```
结果编译的时候打开的开关会与设想的不一样。

### Makefile案例

```makefile
#! Makefile

SRCS := PAPI_flops.c
OBJECTS := $(patsubst %.c, %.o, $(SRCS))
STATIC_LIB := /usr/local/lib/libpapi.a
INCLUDE_DIR := -I/usr/local/include
CC := gcc


all: PAPI_flops

PAPI_flops: $(OBJECTS)
     $(CC) -O0 $< $(STATIC_LIB) -o $@

$(OBJECTS): $(SRCS)
     $(CC) $(INCLUDE_DIR) -O0 -c $<

test:
     echo "----Running the PAPI_flops-----"
     @./PAPI_flops

clean:
     rm -rf PAPI_flops
     rm -rf *.o
```


### makefile中调用.a库的编写

```makefile
#!Makefile
CC = g++

TINYCV_DIR = /home/cwl/TinyCV
TINYCV_INCLUDE_DIR = $(TINYCV_DIR)/include
LIB_DIR = $(TINYCV_DIR)/build

CXX_FLAG = -O3 -std=c++11 -Wall -Werror -fPIC

all: main

main: main.o
  $(CC) $< $(CXX_FLAG) -I$(TINYCV_INCLUDE_DIR) -L$(LIB_DIR) -ltinycv -o $@

main.o: main.cpp
  $(CC) $(CXX_FLAG) -I$(TINYCV_INCLUDE_DIR) -c $<

clean:
  rm -f *.o main
```

需要注意的是下面这句中`$<`是指输入文件main.o，此处紧跟gcc
```
main: main.o
  $(CC) $< $(CXX_FLAG) -I$(TINYCV_INCLUDE_DIR) -L$(LIB_DIR) -ltinycv -o $@
```
但是如果变为如下情形，就会出现后面中的错误
```
main: main.o
  $(CC) $(CXX_FLAG) -I$(TINYCV_INCLUDE_DIR) -L$(LIB_DIR) -ltinycv -o $@  $<
```
错误：
```
caowenlong@Server-NF5280M3:~/Test$ make
g++ -O3 -std=c++11 -Wall -Werror -fPIC -I/home/cwl/TinyCV/include -c main.cpp
g++ -O3 -std=c++11 -Wall -Werror -fPIC -I/home/cwl/TinyCV/include -L/home/cwl/TinyCV/build -ltinycv -o main main.o
main.o：在函数‘main’中：
main.cpp:(.text.startup+0x3b)：对‘tinycv::imread(std::string const&, int)’未定义的引用
main.cpp:(.text.startup+0x43)：对‘tinycv::Mat<unsigned char>::Mat()’未定义的引用
main.cpp:(.text.startup+0x63)：对‘double tinycv::threshold<unsigned char>(tinycv::Mat<unsigned char> const&, tinycv::Mat<unsigned char>&, double, double, int)’未定义的引用
collect2: error: ld returned 1 exit status
make: *** [main] 错误 1
```

### makefile中的全局自变量

`$@`目标文件名
`@^`所有前提名，除副本
`@＋`所有前提名，含副本
`@＜`一个前提名
`@？`所有新于目标文件的前提名
`@*`目标文件的基名称

### 是否输出执行过程

```
#! Makefile
SAMPLE_ENABLE ?= 1

ifeq ($(SAMPLE_ENABLE), 1)
	EXEC ?= @echo "[@]"
endif

target: target2
	echo "hehe, this is target"

target2:
	echo "this is target2"
clean:
	rm -rf out.o
```

- [more samples ](https://github.com/cwlseu/recipes/tree/master/makepractise)

## CMake

### CMake 入门案例
```python
PROJECT(sdk_common_samples)
cmake_minimum_required(VERSION 3.0)

# 查找已经安装的包
FIND_PACKAGE(OpenCV 2)

# SET 指令的语法是:
# SET(VAR [VALUE] [CACHE TYPE DOCSTRING [FORCE]])

SET(
	SDK_COMMON_INCLUDE_DIR
	${CMAKE_CURRENT_SOURCE_DIR}/../../include
	CACHE PATH
	"SDK_COMMON HEADER FILE PATH"
)

# MESSAGE 指令的语法是:
# MESSAGE([SEND_ERROR | STATUS | FATAL_ERROR] "message to display" ...)
# 这个指令用于向终端输出用户定义的信息,包含了三种类型:
# SEND_ERROR,产生错误,生成过程被跳过。
# SATUS ,输出前缀为 — 的信息。
# FATAL_ERROR,立即终止所有 cmake 过程.

MESSAGE("Find libs in ${SDK_COMMON_LIB_DIR}")

# INCLUDE_DIRECTORIES,其完整语法为:
# INCLUDE_DIRECTORIES([AFTER|BEFORE] [SYSTEM] dir1 dir2 ...)
# 这条指令可以用来向工程添加多个特定的头文件搜索路径,路径之间用空格分割,如果路径
# 中包含了空格,可以使用双引号将它括起来,默认的行为是追加到当前的头文件搜索路径的
# 后面,你可以通过两种方式来进行控制搜索路径添加的方式:
# 1,CMAKE_INCLUDE_DIRECTORIES_BEFORE,通过 SET 这个 cmake 变量为 on,可以
# 将添加的头文件搜索路径放在已有路径的前面。
# 2,通过 AFTER 或者 BEFORE 参数,也可以控制是追加还是置前。
INCLUDE_DIRECTORIES(
	${PROJECT_SOURCE_DIR}
	${SDK_COMMON_INCLUDE_DIR}
	${OpenCV_INCLUDE_DIRS}
)

# 添加链接库的文件夹路径
LINK_DIRECTORIES(${SDK_COMMON_LIB_DIR})

# set最长用的方法，就像shell中export一个变量一样
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -Wall -O2 -std=gnu++0x")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -Wall -O2 -std=gnu++0x")

# 查找在相对路径下与*.cpp所匹配的模式的所有文件，保存到变量samples中
FILE(GLOB samples ${PROJECT_SOURCE_DIR}/*.cpp)

# 针对samples中的所有元素进行操作
FOREACH (sample ${samples})
	STRING(REGEX MATCH "[^/]+$" sample_file ${sample})
	STRING(REPLACE ".cpp" "" sample_basename ${sample_file})
	ADD_EXECUTABLE(test_${sample_basename} ${sample})
	# 添加执行时的需要链接的lib： common OpenCV_Libs
	TARGET_LINK_LIBRARIES(test_${sample_basename}
	sdk_common ${OpenCV_LIBS})
	# 另外，如果不是再window下的话需要添加线程库 -lpthread
	if (NOT WIN32)
		TARGET_LINK_LIBRARIES(test_${sample_basename} pthread)
	endif()
	
	INSTALL(TARGETS test_${sample_basename} DESTINATION ${CMAKE_CURRENT_SOURCE_DIR}/bin)
ENDFOREACH() # foreach 结束

```

## windows指定编译器

```cmake
cmake \
-DCMAKE_MODULE_PATH:PATH=Y:\\cmake  -DCMAKE_CONFIGURATION_TYPES=release \
-DCMAKE_INSTALL_PREFIX=C:\\cygwin\\data\\windows-x86_64\\test \
-G "Visual Studio 12 Win64" -T "v120_xp" ..

cmake -G"Visual Studio 12 2013" -A x64 -DCMAKE_BUILD_TYPE=RELEASE ..
cmake -G"Visual Studio 12 2013" -A Win32 -DCMAKE_BUILD_TYPE=RELEASE ..
```
有的时候在windows下，buildtype为debug和release表现不同，而且概率还是比较高的。

### 官网提供的入门教程中的案例

```python
cmake_minimum_required (VERSION 2.6)
project (Tutorial)

# should we use our own math functions?
option (USE_MYMATH 
  "Use tutorial provided math implementation" ON) 

# The version number.
set (Tutorial_VERSION_MAJOR 1)
set (Tutorial_VERSION_MINOR 0)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file (
  "${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
  "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  )

# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h
include_directories("${PROJECT_BINARY_DIR}")


if (USE_MYMATH)
  include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
  add_subdirectory (MathFunctions)
  set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif (USE_MYMATH)

# add the executable
add_executable(Tutorial main.cpp)
target_link_libraries (Tutorial ${EXTRA_LIBS})

# add the install targets
install (TARGETS Tutorial DESTINATION bin)
install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"        
  DESTINATION include)


include(CTest)
# does it sqrt of 25
add_test (TutorialComp25 Tutorial 25)
set_tests_properties (TutorialComp25 PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")
# does it handle negative numbers
#add_test (TutorialNegative Tutorial -25)
#set_tests_properties (TutorialNegative PROPERTIES PASS_REGULAR_EXPRESSION "-25 is 0")

# does it handle small numbers
add_test (TutorialSmall Tutorial 0.0001)
set_tests_properties (TutorialSmall PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 is 0.01")

# does the usage message work?
add_test (TutorialUsage Tutorial)
set_tests_properties (TutorialUsage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

#define a macro to simplify adding tests, then use it
macro (do_test arg result)
  add_test (TutorialComp${arg} Tutorial ${arg})
  set_tests_properties (TutorialComp${arg}
  PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro (do_test)

do_test (81 "81 is 9")
# do a bunch of result based tests
do_test (25 "25 is 5")
do_test (-25 "-25 is 0")
```

### CMake 特点
* 跨平台，并且可以生成响应的编译配置文件，如在linux平台下生成makefile,在苹果下生成xcode,在windows下可以生成MSVC的工程文件
* 开源，使用类BSD许可发布
* 简化管理大型项目
* 简化编译构建过程和编译过程cmake + make
* 可拓展，可以编写特定功能的模块

### CMake问题

* cmake编写的过程实际上是编程，每个目录一个CMakeLists.txt，使用cmake语言和语法
* 一些拓展可以使用，但是配合起来可能不是很理想
* 针对大型项目，如果项目比较小，还是直接编写makefile比较好

### 定义变量

1. 命令行中
`cmake -DCUDA_USE_STATIC_CUDA_RUNTIME=1 ..`
2. [cmake 中set的使用](https://cmake.org/cmake/help/v3.10/command/set.html?highlight=set)

```cmake
# 普通变量定义
SET(DIDBUILD_TARGET_OS LINUX)
# 强制覆盖
SET(CUDA_USE_STATIC_CUDA_RUNTIME OFF CACHE BOOL "fix cuda compiling error" FORCE)
# 有则忽略，否则定义变量
SET(DIDBUILD_TARGET_ARCH X86_64 CACHE STRING "default arch is x86_64")
# 设置环境变量
SET(ENV{LD_LIBRARY_PATH} /usr/local/lib64)
```

### [字符串处理](https://cmake.org/cmake/help/v3.10/command/string.html?highlight=string#command:string)

`STRING(FIND $Origin_str $substr $target_str)`

此外，`FIND`,`REPLACE`,`REGEX MATCH`，`APPEND`
`string(CONCAT <output variable> [<input>...])`

Concatenate all the input arguments together and store the result in the named output variable.

`string(TOLOWER <string1> <output variable>)`

Convert string to lower characters.

`string(LENGTH <string> <output variable>)`

Store in an output variable a given string’s length.

`string(SUBSTRING <string> <begin> <length> <output variable>)`

Store in an output variable a substring of a given string. If length is -1 the remainder of the string starting at begin will be returned. If string is shorter than length then end of string is used instead.

`string(STRIP <string> <output variable>)`

Store in an output variable a substring of a given string with leading and trailing spaces removed.
```cmake
string(COMPARE LESS <string1> <string2> <output variable>)
string(COMPARE EQUAL <string1> <string2> <output variable>)
string(<HASH> <output variable> <input>)
```
Compute a cryptographic hash of the input string. The supported `<HASH>` algorithm names are: 很多

### `STREQUAL`

### `make VERBOSE=1` 

可以将cmake中定义的变量打印

### Object Libraries

The OBJECT library type is also not linked to. It defines a non-archival collection of object files resulting from compiling the given source files. The object files collection can be used as source inputs to other targets:

```cmake
add_library(archive OBJECT archive.cpp zip.cpp lzma.cpp)
add_library(archiveExtras STATIC $<TARGET_OBJECTS:archive> extras.cpp)
add_executable(test_exe $<TARGET_OBJECTS:archive> test.cpp)
```

OBJECT libraries may only be used locally as sources in a buildsystem – they may not be installed, exported, or used in the right hand side of `target_link_libraries()`. They also may not be used as the `TARGET` in a use of the `add_custom_command(TARGET)` command signature.

Although object libraries may not be named directly in calls to the `target_link_libraries()` command, they can be “linked” indirectly by using an Interface Library whose `INTERFACE_SOURCES` target property is set to name `$<TARGET_OBJECTS:objlib>`.

###  ExternalProject，通过url配置依赖第三方库

> cmake/DownloadGoogleBenchmark.cmake 

```cmake
INCLUDE(ExternalProject)
ExternalProject_Add(googletest
	URL https://github.com/google/googletest/archive/release-1.8.0.zip
	URL_HASH SHA256=f3ed3b58511efd272eb074a3a6d6fb79d7c2e6a0e374323d1e6bcbcc1ef141bf
	SOURCE_DIR "${CONFU_DEPENDENCIES_SOURCE_DIR}/googletest"
	BINARY_DIR "${CONFU_DEPENDENCIES_BINARY_DIR}/googletest"
	CONFIGURE_COMMAND ""
	BUILD_COMMAND ""
	INSTALL_COMMAND ""
	TEST_COMMAND ""
)
```

> 主CMakeLists.txt中的使用

```cmake
IF(PTHREADPOOL_BUILD_BENCHMARKS AND NOT DEFINED GOOGLEBENCHMARK_SOURCE_DIR)
     MESSAGE(STATUS "Downloading Google Benchmark to     ${CONFU_DEPENDENCIES_SOURCE_DIR}/googlebenchmark (define GOOGLEBENCHMARK_SOURCE_DIR to avoid it)")
     # 添加其他依赖路径
     CONFIGURE_FILE(cmake/DownloadGoogleBenchmark.cmake "${CONFU_DEPENDENCIES_BINARY_DIR}/googlebenchmark-download/CMakeLists.txt")
     EXECUTE_PROCESS(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
     WORKING_DIRECTORY "${CONFU_DEPENDENCIES_BINARY_DIR}/googlebenchmark-download")
     EXECUTE_PROCESS(COMMAND "${CMAKE_COMMAND}" --build .
     WORKING_DIRECTORY "${CONFU_DEPENDENCIES_BINARY_DIR}/googlebenchmark-download")
     SET(GOOGLEBENCHMARK_SOURCE_DIR "${CONFU_DEPENDENCIES_SOURCE_DIR}/googlebenchmark" CACHE STRING "Google Benchmark source directory")
ENDIF()
```
## CMakeLists中的高级用法
```cmake
INSTALL(TARGETS libdeepindeed
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib)
```
- [cmake install target](https://cmake.org/cmake/help/v3.1/command/install.html#installing-targets)

- 库之间的符号继承等

## 参考资料

- [1] [cmake buildsystem文档，主要关于target_property, target_include_directories,target_link_libraries,set_target_properties](https://cmake.org/cmake/help/v3.10/manual/cmake-buildsystem.7.html)
- [2] [ExternalProject文档](https://cmake.org/cmake/help/v3.0/module/ExternalProject.html)
- [3] [CMake Practice](https://app.yinxiang.com/shard/s40/res/ecb203bd-889b-4eb3-8ee6-d0b0e88765f6/CMake%20Practice.pdf?search=Cmake)
- [4] [Makefile中文简明教程(陈皓)](https://app.yinxiang.com/shard/s40/res/67a665d8-3622-49d1-ac10-2b21c8f29277/Makefile%E4%B8%AD%E6%96%87%E6%95%99%E7%A8%8B.pdf?search=Cmake)
- [5] [CMake如何查找链接库---find_package的使用方法](https://blog.csdn.net/u011092188/article/details/61425924)
- [6]. 练习CMake的项目: https://github.com/cwlseu/brick

## CMake manual

- [cmake buildsystem](https://cmake.org/cmake/help/v3.12/manual/cmake-buildsystem.7.html)
- cmake packages
  - [creating-packages](https://cmake.org/cmake/help/v3.12/manual/cmake-packages.7.html#creating-packages)
  - [cmaketest sample](https://github.com/cwlseu/codefeeling/tree/master/cmaketest/createpackage)