---
title: CMake&Meson 边学边记
date: 2021-04-10 20:32:42
tags: [CMake,Meson]
categories: Tools
description: 学习生产力工具CMake、Meson
---

[TOC]

# CMake笔记

**一开始是记录CMake的学习，7月份实习用到了Meson，故也记录在了这篇文章中。**

##　安装

### Windows安装

* 网址：https://cmake.org/download/

###  Linux安装

* 直接安装 `apt install cmake`

* 网址：https://cmake.org/download/     下载对应版本的CMake（32位或者64位）

* 输入以下命令进行解压

  `	tar -zxvf cmake-3.10.0-rc4-Linux-x86_64.tar.gz`

* 把解压后的目录改名为：cmake

  `mv cmake-3.10.0-rc4-Linux-x86_64 cmake`

* 安装完毕，命令行输入：`cmake --version`检测是否安装成功

## CMake初探

**基本都在我的阿里云服务器上倒腾，环境为Ubuntu 16.04**

### CMake基础知识

* 最低版本

  * `CMakeLists.txt`的第一行都会写：`cmake_minimum_required(VERSION 3.1)`，该命令指定了CMake的最低版本是3.1

  * 命令名称`cmake_minimum_required`不区分大小写

  * 设置版本范围：`cmake_minimum_required(VERSION 3.1...3.12)`

  * 判断CMake版本：

    ```
    #该命令表示：如果CMake版本小于3.12，则if块将为true，然后将设置为当前CMake版本
    #如果CMake版本高于3.12，if块为假，cmake_minimum_required将被正确执行
    if(${CMAKE_VERSION} VERSION_LESS 3.12)     
        cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}) 
    endif() 
    ```

  * 注意：如果需要支持非命令行Windows版本则需在上面的if判断加上else分支，如下

    ```
    cmake_minimum_required(VERSION 3.1)
    if(${CMAKE_VERSION} VERSION_LESS 3.12)
        cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
    else()
        cmake_policy(VERSION 3.12)
    endif()
    ```

* 设置生成项目名称
  * 命令：`project（MyProject）`,表示生成的工程名字叫做：`MyProject`
  
  * 命令还可以标识项目支持的语言，写法：`project（MyProject[C] [C++]）`,不过通常将后面的参数省掉，因为默认支持所有语言
  
  * 使用该指令之后系统会自动创建两个变量：`<projectname>_BINARY_DIR`  二进制文件保存路径、`<projectname>_SOURCE_DIR`  源代码路径
  
  * 执行`project(MyProject)`，就是定义了一个项目的名称为`MyProject`，对应的就会生成两个变量：`_BINARY_DIR`和`_SOURCE_DIR`，但是`cmake`中其实已经有两个预定义的变量：`PROJECT_BINARY_DIR` 和 `PROJECT_SOURCR_DIR`
  
  * 关于两个变量是否相同，涉及到是内部构建还是外部构建
  
    * 内部构建
  
      ```
      cmake ./
      make
      ```
  
    * 外部构建
  
      ```
      mkdir build
      cd ./build
      cmake ../
      make
      ```
  
    * 内部构建和外部构建的不同在于：`cmake `的工作目录不同。内部构建会将`cmake`生成的中间文件和可执行文件放在和项目同一目录；外部构建的话，中间文件和可执行文件会放在`build`目录
  
    * `PROJECT_SOURCE_DIR`和`_SOURCE_DIR`无论内部构建还是外部构建，指向的内容都是一样的，都指向工程的根目录
  
    * `PROJECT_BINARY_DIR`和`_BINARY_DIR`指向的相同内容，内部构建的时候指向`CMakeLists.txt`文件的目录，外部构建指向`target`编译的目录
  
* 生成可执行文件

  * 语法：`add_executable(exename srcname)`

    > * exename:生成的可执行文件的名字
    > * srcname:原来的源文件

  * 该命令指定生成可执行文件的名字以及指出需要依赖的源文件的文件名

  * 获取文件路径中的所有源文件

    * 命令：`aux_sourcr_directory(<dir> <variable>)`
    * 例子：`aux_sourcr_directory(. DIR_SRCS)`，将当前目录下的源文件名字存放到变量`DIR_SRCS`里面 ，如果源文件比较多，直接用`DIR_SRCS`变量即可

  * 生成可执行文件：`add_executable(Demo ${DIR_SRCS})`，将生成的可执行文件命名为：`Demo`

* 生成`lib`库

  * 命令：`add_library(libname [SHARED|STATIC|MODULE] [EXCLUDE_FROM_ALL] source1 source2 ... sourceN)`

    > * `libname`:生成的库文件的名字
    > * `[SHARED|STATIC|MODULE]`：生成库文件的类型（动态库|静态库|模块）
    > * `[EXCLUDE_FROM_ALL]`：有这个参数表示该库不会被默认构建
    > * `source2 ... sourceN`：生成库依赖的源文件，如果源文件比较多，可以使用
  > * `aux_sourcr_directory`命令获取路径下所有源文件，具体章节参见：`CMake`初探->生成可执行文件->获取路径中所有源文件
  
  * 例子：`add_library(ALib SHARE alib.cpp)`

```
add_executable(demo demo.cpp) # 生成可执行文件
add_library(common STATIC util.cpp) # 生成静态库
add_library(common SHARED util.cpp) # 生成动态库或共享库
```

> `add_library` 默认生成是静态库，通过以上命令生成文件名字，
>
> - 在 Linux 下是：
>   demo
>   libcommon.a
>   libcommon.so
> - 在 Windows 下是：
>   demo.exe
>   common.lib
>   common.dll

* 添加头文件目录

  * 命令1：`target_include_directories(<target> [SYSTEM] [BEFORE]   <INTERFACE|PUBLIC|PRIVATE> [items1...]   [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])`

    >当我们添加子项目之后还需要设置一个`include`路径，例子：
    >eg:`target_include_directories(RigelEditor PUBLIC ./include/rgeditor)`，表示给`RigelEditor` 这个子项目添加一个库文件的路径

  * 命令2：`include_directories([AFTER|BEFORE] [SYSTEM] dir1 [dir2 …])`

    >参数解析：
    >
    >* [AFTER|BEFORE]：指定了要添加路径是添加到原有列表之前还是之后
    >* [SYSTEM]：若指定了`system`参数，则把被包含的路径当做系统包含路径来处理
    >* dir1 [dir2 …]把这些路径添加到`CMakeLists`及其子目录的`CMakeLists`的头文件包含项目中
    >  相当于`g++`选项中的-l的参数的作用
    >* 举例：`include_directories("/opt/MATLAB/R2012a/extern/include")`

  - 两条指令的作用都是讲将`include`的目录添加到目标区别在于`include_directorie`s是`CMake`编译所有目标的目录进行添加，`target_include_directories`是将`CMake`编译的指定的特定目标的包含目录进行添加

* 添加需要链接的库文件路径

  * 命令1:`target_link_libraries(<target> [item1 [item2 [...]]] [[debug|optimized|general] <item>] ...)`

    >- 作用：为给定的目标设置链接时使用的库（设置要链接的库文件的名称）
    >- eg:target_link_libraries(MyProject a b.a [c.so](http://c.so/))    //将若干库文件链接到hello中，target_link_libraries里的库文件的顺序符合gcc/g++链接顺序规则，即：被依赖的库放在依赖他的库的后面，如果顺序有错，链接将会报错
    >- 关键字：debug对应于调试配置
    >- 关键字：optimized对应于所有其他的配置类型
    >- 关键字：general对应于所有的配置（该属性是默认值）

  * 命令2：`link_libraries`

    >- 作用：给当前工程链接需要的库文件（全路径）
    >- eg:`link_libraries(("/opt/MATLAB/R2012a/bin/glnxa64/libeng.so")`//必须添加带名字的全路径

  * 区别：`target_link_libraries`可以给工程或者库文件设置其需要链接的库文件，而且不需要填写全路径，但是`link_libraries`只能给工程添加依赖的库，而且必须添加全路径

  * 添加需要链接的库文件目录

    >- 命令：link_directories（添加需要链接的库文件目录）
    >
    >- 语法：link_directories(directory1 directory2 ...)
    >
    >- 例子：link_directories("/opt/MATLAB/R2012a/bin/glnxa64")

  * 指令的区别：指令的前缀带`target`，表示针对某一个目标进行设置，必须指明设置的目标；`include_directories`是在编译时用，指明`.h`文件的路径；`link_directoeies`是在链接时用的，指明链接库的路径；`target_link_libraries`是指明链接库的名字，也就是具体谁链接到哪个库。`link_libraries`不常用，因为必须指明带文件名全路径

* 控制目标属性

  * 以上的几条命令的区分都是：是否带`target`前缀，在`CMake`里面，一个`target`有自己的属性集，如果我们没有显示的设置这些`target`的属性的话，`CMake`默认是由相关的全局属性来填充`target`的属性，我们如果需要单独的设置`target`的属性，需要使用命令：`set_target_properties()`

  * 命令格式:

    >set_target_properties(target1 target2 ...
    >PROPERTIES
    >属性名称1  值
    >属性名称2  值
    >...
    >)

  * 控制编译选项的属性是：`COMPILE_FLAGS`

  * 控制链接选项的属性是：`LINK_FLAGS`

  * 控制输出路径的属性：`EXECUTABLE_OUTPUT_PATH`（exe的输出路径）、`LIBRARY_OUTPUT_PATH`（库文件的输出路径）

  * 举例：

    >set_target_properties(exe
    >PROPERTIES
    >LINK_FLAGS          -static
    >LINK_FLAGS_RELEASE  -s
    >)

    这条指令会使得`exe`这个目标在所有的情况下都采用`-static`选项，而且在`release build`的时候` -static -s `选项。但是这个属性仅仅在`exe`这个`target`上面有效

### Cmake构建示例

经典的`helloworld`必须出现啊，首先创建一个`main.cpp`包含源的文件

```c++
#include <iostream>
using namespace std;

int main(int parameter_size, char **parameter)
{
    int a = 1 + 3;
    int b = a + 3;
    cout << "hello word  " << parameter_size << "   " << endl;
    return 0;
}
```

然后我们创建一个 `CMakeLists.txt`同一个目录中

```
cmake_minimum_required(VERSION 3.5)
#项目名称
project(test_cmaka)
#代码路径
aux_source_directory(. DIR_TOOT_SRCS)
#dubug 模式
set (CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -g")
#将可执行文件放入bin目录
set( EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)
#生成可执行的文件
add_executable(main ${DIR_TOOT_SRCS})
```

文件目录 如下

```
|-- test_cmake
    |-- main.cpp
    |-- CMakeLists.txt
```

现在准备构建我们的应用程序，导航到文件目录下，创建`build`文件夹

```
$ mkdir build
```

进入`build`目录，执行外部构建

```
$ cd build
# cmake ../
```

输出

```
-- The C compiler identification is GNU 7.5.0
-- The CXX compiler identification is GNU 7.5.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /root/share/C++/test_cmake/build
```

再看看目录下的文件

```
CMakeCache.txt  CMakeFiles  cmake_install.cmake  Makefile
```

可以看到成功生成了Makefile，还有一些cmake运行时自动生成的文件
然后在终端下输入make并回车,输出以下

```
Scanning dependencies of target main
[ 50%] Building CXX object CMakeFiles/main.dir/main.cpp.o
[100%] Linking CXX executable ../bin/main
[100%] Built target main
```

此时，文件目录如下

```
|-- test_cmake
    |-- bin
    |-- build
    |-- main.cpp
    |-- CMakeLists.txt
```

生成的可执行文件在`bin`目录下，导航到该目录下

```
cd ../bin
./main
```

产生预期的输出

```
hello word  1 
```

`github`上有不错的`CMake——examples`  链接：https://github.com/ttroy50/cmake-examples

# Meson笔记

## 安装

### Linux安装

* Meson基于Python3运行，要求Python版本3.5以上

* 安装依赖 `ninja-build`

  ```
  sudo apt-get install ninja-build
  ```

* 安装`Meson`

  ```
  pip3 install meson #root      
  
  pip3 install --user meson #user 官方推荐
  ```

* 也从`git`下载安装

  ```
  git clone https://github.com/mesonbuild/meson.git /path/to/sourcedir
  ```

### Meson构建示例

依然经典的`helloworld`，首先创建一个`main.c`包含源的文件

```c
# main.c
#include <stdio.h>

int main(int argc, char **argv) {
  printf("Hello there.\n");
  return 0;
}
```

然后我们创建一个 `meson.build`同一个目录中调用的文中

```
project('tutorial', 'c')
executable('main', 'main.c')
```

文件目录如下

```
|-- test_meson
    |-- main.c
    |-- meson.build
```

现在准备构建我们的应用程序，导航到文件

```
#构建程序
$ meson build
```

创建一个单独的构建目录`build`来保存所有编译器输出。`Meson` 与其他一些构建系统的不同之处在于它不允许源代码构建。您必须始终创建一个单独的构建目录。常见的约定是将默认构建目录放在顶级源目录的子目录中

当 `Meson` 运行时，它会打印以下输出

```
The Meson build system
Version: 0.58.1
Source dir: /root/share/C++/test_meson
Build dir: /root/share/C++/test_meson/build
Build type: native build
Project name: tutorial
Project version: undefined
C compiler for the host machine: cc (gcc 7.5.0 "cc (Ubuntu 7.5.0-3ubuntu1~18.04) 7.5.0")
C linker for the host machine: cc ld.bfd 2.30
Host machine cpu family: x86_64
Host machine cpu: x86_64
Build targets in project: 1
```

现在已经准备好构建我们的代码了

```
$ cd buildd
$ ninja
```

如果您的 `Meson` 版本高于 `0.55.0`，您可以使用新的后端不可知构建命令

```
$ cd build
$ meson compile
```

一旦构建了可执行文件，我们就可以运行它

```
./main
```

这会产生预期的输出

```
Hello there.
```



​	



# 参考



https://mubu.com/doc/t1VDCEn4O0#o-17f166a665726b18d

https://cmake.org/documentation

https://cmake.org/cmake/help/latest/guide/tutorial/index.html

https://mesonbuild.com/







