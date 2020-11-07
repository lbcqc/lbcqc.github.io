---
layout: post
title: CMake：基本用法
categories: [Build Tools]
description: some word here
keywords: keyword1, keyword2
---

工作中多处使用到 CMAKE，以前学习过 Makefile，但是对 CMake 了解很少，借此机会，系统的学习整理下 CMAKE 并记录下来。  

## 一、概述

CMake是一个管理构建过程的开源软件，支持多平台的、多编译器、多构建系统。  

### （一）、兼容性

- 平台：Linux，MaxOS，Windows，Cygwin
- 编译器：Gcc(默认)，Clang
- 构建系统：Makefile(默认)，ninja，Qt Creator，Xcode，Microsoft Visual Studio

### （二）、文档

- wiki：<https://en.wikipedia.org/wiki/CMake>
- 官方文档：<https://cmake.org/documentation/>
- cmake-example：<https://github.com/ttroy50/cmake-examples.git>

推荐官方文档中的 Commands 手册，里面有对每个命令的解释：<https://cmake.org/cmake/help/v3.19/manual/cmake-commands.7.html>

### （三）、其它

1. CMake和Makefile的区别？

    CMake可以使用Makefile作为构建工具来管理构建过程，同时CMake还支持其它构建系统，比如ninja。
    CMake相比于Makefile更容易处理跨平台问题，更容易管理第三方库、导入包等等。

2. CMake和Automake的比较？

    两者都是通过Makefile等构建系统工作，都是跨平台的，但是CMake出现的更晚，功能更加丰富，构建过程更高效。

## 二、基本变量

| Variable           | Info                                                         |
| :----------------- | :----------------------------------------------------------- |
| PROJECT_NAME       | The name of the project set by the current `project()`.      |
| CMAKE_PROJECT_NAME | the name of the first project set by the `project()` command, i.e. the top level project. |
| PROJECT_SOURCE_DIR | The source director of the current project.                  |
| PROJECT_BINARY_DIR | The build directory for the current project.                 |
| name_SOURCE_DIR    | The source directory of the project called "name". In this example the source directories created would be `sublibrary1_SOURCE_DIR`, `sublibrary2_SOURCE_DIR`, and `subbinary_SOURCE_DIR` |
| name_BINARY_DIR    | The binary directory of the project called "name". In this example the binary directories created would be `sublibrary1_BINARY_DIR`, `sublibrary2_BINARY_DIR`, and `subbinary_BINARY_DIR` |

## 三、基础用例

### （一）、构建可执行文件

最简单的情况下，没有头文件，没有第三方库，只需要将所有的源文件编译链接生成一个可执行文件，如下所示：

- `add_executable`

    ```bash
    add_executable(<name> [WIN32] [MACOSX_BUNDLE]
               [EXCLUDE_FROM_ALL]
               [source1] [source2 ...])
    ```

    添加一个可执行文件目标 `name` ，去构建给出的所有 `source` 源文件列表。

    **WIN32** ：构建 WIN32 可执行文件。

    **MACOSX_BUNDLE** ：构建 MACOSX 可执行文件。

    **EXCLUDE_FROM_ALL** ： 如果设置了该选项，`all` 目标将不包含此目标。

    [详细介绍](https://cmake.org/cmake/help/v3.19/command/add_executable.html)

- 工程目录结构

    [example-source](https://github.com/ttroy50/cmake-examples/tree/master/01-basic/A-hello-cmake)

    ```bash
    .
    ├── CMakeLists.txt
    ├── main.cpp
    ```

- CMakeLists.txt

    ```CMake
    # 设置cmake最小版本
    cmake_minimum_required(VERSION 3.5)

    # 工程名（注意，test已经被CMake测试组件CTest当作预留工程名）
    project (hello_cmake)

    # 添加可执行文件
    add_executable(hello_cmake main.cpp)
    ```

- 其它

    不建议使用一个变量来保存源文件列表，这有时会导致修改文件时，不触发 CMake 重新构建，如下所示：

    ```CMake
    # Create a sources variable with a link to all cpp files to compile
    set(SOURCES
        src/Hello.cpp
        src/main.cpp
    )

    # Add an executable with the above sources
    add_executable(hello_headers ${SOURCES})
    ```

    最好直接在 add_executable 后面追加源文件列表。

### （二）、包含头文件

- `target_include_directories`

    ```bash
    target_include_directories(<target> [SYSTEM] [BEFORE]
        <INTERFACE|PUBLIC|PRIVATE> [items1...]
        [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
    ```

    指定编译目标 `target` 时需要使用到的头文件目录列表 `items1 items2 ...` ，使用该指令时，目标应该已经被提前声明。

    | 属性          | 描述                                                         |
    | ------------- | ------------------------------------------------------------ |
    | **SYSTEM**    | 指定该目录作为系统的 include 目录。                          |
    | **BEFORE**    | 指定该头文件列表是往该属性的前面添加，而不是往末尾添加。<br />这里注意：一个目标依赖的头文件并不局限于直接指定的，可能还会间接使用到引用的第三方库的头文件目录。 |
    | **PUBLIC**    | 自身需要包含，其它引用了该 `target` 的目标也需要包含。       |
    | **INTERFACE** | 自身不需要包含，其它引用了该 `target` 的目标需要包含。       |
    | **PRIVATE**   | 自身需要包含，其它引用了该 `target` 的目标不需要包含。       |

    对于 `<INTERFACE|PUBLIC|PRIVATE>`，如果没有特殊需求，直接使用 `PUBLIC` 即可。

    [详细介绍](https://cmake.org/cmake/help/v3.19/command/target_include_directories.html)

- 工程目录结构

    [example-source](https://github.com/ttroy50/cmake-examples/tree/master/01-basic/B-hello-headers)

    ```bash
    8675 ± tree
    .
    ├── CMakeLists.txt
    ├── include
    │   └── Hello.h
    ├── README.adoc
    └── src
        ├── Hello.cpp
        └── main.cpp
    
    2 directories, 5 files
    ```

- CMakeLists.txt

    ```CMake
    # Set the minimum version of CMake that can be used
    # To find the cmake version run
    # $ cmake --version
    cmake_minimum_required(VERSION 3.5)

    # Set the project name
    project (hello_headers)

    # Create a sources variable with a link to all cpp files to compile
    set(SOURCES
        src/Hello.cpp
        src/main.cpp
    )

    # Add an executable with the above sources
    add_executable(hello_headers ${SOURCES})

    # Set the directories that should be included in the build command for this target
    # when running g++ these will be included as -I/directory/path/
    target_include_directories(hello_headers
        PRIVATE
            ${PROJECT_SOURCE_DIR}/include
    )
    ```

### （三）、构建静态库

- `add_library`

  ```cmake
  add_library(<name> [STATIC | SHARED | MODULE]
              [EXCLUDE_FROM_ALL]
              [<source>...])
  ```

  添加一个库文件目标，可以为静态库、动态库、模块。

  | 属性             | 描述                                                     |
  | ---------------- | -------------------------------------------------------- |
  | STATIC           | 静态库：在编译链接时加入可执行文件中。                   |
  | SHARED           | 动态库：在运行时加载进可执行文件中。                     |
  | MODULE           | 模块：不会自动加载进可执行文件中，但可以被函数动态导入。 |
  | EXCLUDE_FROM_ALL | 排除在 ALL 目标之外                                      |
  | <source>...      | 生成该库文件的源文件列表                                 |

  如果一个库没有对外展示任何符号，那么其应当被声明为 `MODULE`。

  [详细介绍](https://cmake.org/cmake/help/v3.19/command/add_library.html)

- `target_link_libraries`

  该函数有多个重载实现，这里展示其中两个。

  ```cmake
  target_link_libraries(<target> ... <item>... ...)
  ```

  给目标 <target> 添加依赖库。调用该命令时，<target> 必须提前被 add_executable 或者 add_library 声明。

  如果多次调用该函数给相同的目标 <target> 添加依赖库，将按序追加。

  ```cmake
  target_link_libraries(<target>
                        <PRIVATE|PUBLIC|INTERFACE> <item>...
                       [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
  ```

  | 属性      | 描述                         |
  | --------- | ---------------------------- |
  | PRIVATE   | 只链接库，但不继承它的依赖。 |
  | PUBLIC    | 链接该库，并且继承它的依赖。 |
  | INTERFACE | 不链接该库，只基础它的依赖。 |

  [详细介绍](https://cmake.org/cmake/help/v3.19/command/target_link_libraries.html)

- 工程目录结构

  [example-source](https://github.com/ttroy50/cmake-examples/tree/master/01-basic/C-static-library)

  ```bash
  edit ॐ  ~/workspace/other/cmake-examples/01-basic/C-static-library:(193d19h37m|git@master)
  8675 ± tree
  ├── CMakeLists.txt
  ├── include
  │   └── static
  │       └── Hello.h
  ├── README.adoc
  └── src
      ├── Hello.cpp
      └── main.cpp
  
  3 directories, 5 files
  ```

- CMakeLists.txt

  ```cmake
  cmake_minimum_required(VERSION 3.5)
  
  project(hello_library)
  
  ############################################################
  # Create a library
  ############################################################
  
  #Generate the static library from the library sources
  add_library(hello_library STATIC 
      src/Hello.cpp
  )
  
  target_include_directories(hello_library
      PUBLIC 
          ${PROJECT_SOURCE_DIR}/include
  )
  
  
  ############################################################
  # Create an executable
  ############################################################
  
  # Add an executable with the above sources
  add_executable(hello_binary 
      src/main.cpp
  )
  
  # link the new hello_library target with the hello_binary target
  target_link_libraries( hello_binary
      PRIVATE 
          hello_library
  )
  
  ```

### （四）、构建动态库

​	动态库构建和静态库基本一致，只需将 add_library 函数的 STATIC 改为 SHARED 即可。

- 工程目录结构

  [example-source](https://github.com/ttroy50/cmake-examples/tree/master/01-basic/D-shared-library)

  ```bash
  edit ॐ  ~/workspace/other/cmake-examples/01-basic/D-shared-library:(193d19h47m|git@master)
  8680 ± tree
  ├── CMakeLists.txt
  ├── include
  │   └── shared
  │       └── Hello.h
  ├── README.adoc
  └── src
      ├── Hello.cpp
      └── main.cpp
  
  3 directories, 5 files
  ```

- CMakeLists.txt

  ```cmake
  cmake_minimum_required(VERSION 3.5)
  
  project(hello_library)
  
  ############################################################
  # Create a library
  ############################################################
  
  #Generate the shared library from the library sources
  add_library(hello_library SHARED 
      src/Hello.cpp
  )
  add_library(hello::library ALIAS hello_library)
  
  target_include_directories(hello_library
      PUBLIC 
          ${PROJECT_SOURCE_DIR}/include
  )
  
  ############################################################
  # Create an executable
  ############################################################
  
  # Add an executable with the above sources
  add_executable(hello_binary
      src/main.cpp
  )
  
  # link the new hello_library target with the hello_binary target
  target_link_libraries( hello_binary
      PRIVATE 
          hello::library
  )
  ```