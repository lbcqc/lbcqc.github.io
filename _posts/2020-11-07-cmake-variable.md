---
layout: post
title: CMake：变量
categories: [Build Tools]
description: CMake 变量
keywords: CMake, Variable


---

这篇文章记录一下 CMake 中的变量。

## 一、概述

[详细介绍](https://cmake.org/cmake/help/v3.19/manual/cmake-language.7.html#cmake-language-variables)

变量是基本的存储单元，它们的值总是字符串类型，尽管一些命令将字符串解释为其它类型。`set()` 和 `unset()` 函数用来设置或取消变量的值，有些函数也有设置变量的含义。变量名只能由字母数字`_-`组成。

### （一）、普通变量 & CACHE 变量

变量有动态作用域，set或者unset只是将变量绑定在该作用域内。

| 作用域     | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| Function   | 只在函数和其嵌套内有效，返回之后无效。                       |
| Directory  | 不再Function范围内的set 或 unset将绑定在当前目录下。每个目录 CMakeLists.txt 初始化时都会使用父目录变量值来初始化自身目录的变量。 |
| Persistent | CACHE 变量，区别于上面两个普通变量，独立存储，在构建期内都有效，只能通过 `CACHE` 选项在 set 和 unset 中才能修改它的值。 |

当我们去引用一个变量 `${<variable>}` 时，CMake 首先从 Function 作用域找，找不到的话往 Directory 找，如果还找不到变量，或者变量已经被 unset 了，就往 CACHE 里面找。另外，可以直接通过 `$CACHE{<variable>}` 读取 CACHE 中的值。

`if()` 语句比较特殊，可以直接通过 `<variable>` 去访问普通变量，然后环境变量和 CACHE 变量还是需要通过 `$ENV{<variable>}` or `$CACHE{<variable>}`.去访问。

另外，还可以通过命令行参数指定 `-D<var>=<value>` 创建 CACHE 类型变量，但是这里没办法指定类型，可以使用在 CMakeLists.txt 中用 `set` 去而外指定。

> Note
>
> CMake reserves identifiers that:
>
> - begin with `CMAKE_` (upper-, lower-, or mixed-case), or
> - begin with `_CMAKE_` (upper-, lower-, or mixed-case), or
> - begin with `_` followed by the name of any [`CMake Command`](https://cmake.org/cmake/help/v3.19/manual/cmake-commands.7.html#manual:cmake-commands(7)).

### （二）、环境变量

环境变量和普通变量有如下不同：

- 作用域

  环境变量有全局作用域，不需要被 CACHE。

- 访问

  `$ENV{<variable>}`

- 初始化

  CMake 环境变量的初始值是调用进程时的初始值。可以通过 set 和 unset 来修改环境变量的值, 但是这个修改不会写回调用进程, 构建完成之后的测试进程也不可见该修改。

### （三）、列表

尽管所有的变量值都是字符串，但是有些字符串被 `;` 分割为列表，通常字符串中不应该有 `;` 这个字符，这个字符已经被赋予特殊意义。

```cmake
set(srcs a.c b.c c.c) # sets "srcs" to "a.c;b.c;c.c"
set(x a "b;c") # sets "x" to "a;b;c", not "a;b\;c"
```

## 二、set

[详细信息](https://cmake.org/cmake/help/v3.19/command/set.html)

### （一）、普通变量

```cmake
set(<variable> <value>... [PARENT_SCOPE])
```

在当前 Function 或者 Directory 作用域设置 `<variable>` 为 `<value>` 值。

如果设置了选项 `PARENT_SCOPE`，那么该变量将被设置到上一个 Function  或者 Directory，在当前的作用域内其不变。

### （二）、CACHE 变量

```cmake
set(<variable> <value>... CACHE <type> <docstring> [FORCE])
```

`type` 可以为下列值:

| type     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| BOOL     | Boolean `ON/OFF` value. [`cmake-gui(1)`](https://cmake.org/cmake/help/v3.19/manual/cmake-gui.1.html#manual:cmake-gui(1)) offers a checkbox. |
| FILEPATH | Path to a file on disk. [`cmake-gui(1)`](https://cmake.org/cmake/help/v3.19/manual/cmake-gui.1.html#manual:cmake-gui(1)) offers a file dialog. |
| PATH     | Path to a directory on disk. [`cmake-gui(1)`](https://cmake.org/cmake/help/v3.19/manual/cmake-gui.1.html#manual:cmake-gui(1)) offers a file dialog. |
| STRING   | A line of text. [`cmake-gui(1)`](https://cmake.org/cmake/help/v3.19/manual/cmake-gui.1.html#manual:cmake-gui(1)) offers a text field or a drop-down selection if the [`STRINGS`](https://cmake.org/cmake/help/v3.19/prop_cache/STRINGS.html#prop_cache:STRINGS) cache entry property is set. |
| INTERNAL | A line of text. [`cmake-gui(1)`](https://cmake.org/cmake/help/v3.19/manual/cmake-gui.1.html#manual:cmake-gui(1)) does not show internal entries. They may be used to store variables persistently across runs. Use of this type implies `FORCE`. |

`docstring` 必须为一行文本，用来简要介绍该变量，在 `cmake-gui` 中将会显示。

`FORCE` 覆盖原有值，并强制清空所有普通变量重新计算新值。

### （三）、环境变量

```cmake
set(ENV{<variable>} [<value>])
```

设置环境变量为新的值，通过 `$ENV{<variable>}` 访问将会返回新值。

如果 `ENV{<variable>}` 之后没有给出参数，或者 `<value>` 是一个空字符串，那么该命令将清除该环境变量的任何现有值。

## 三、unset

[详细信息](https://cmake.org/cmake/help/v3.19/command/unset.html)

### （一）、普通变量 & CACHE 变量

```cmake
unset(<variable> [CACHE | PARENT_SCOPE])
```

从当前作用域中移除普通变量或者 CACHE 变量，使其重新变为未定义。

注意，如果想设置一个普通变量 `$<var>` 为空，可以使用 `set(<var> "") ` 强制设置。因为 unset 掉一个普通变量之后，可能 CACHE 有同名变量，`$<var>` 将会获取到隐藏的 CACHE 变量值。

### （二）、环境变量

```cmake
unset(ENV{<variable>})
```

移除一个环境变量，`$ENV{<variable>}` 将会返回空字符串。

注意，此操作只影响 CMake 进程，不影响机器环境变量。

## 四、set_property

[详细信息](https://cmake.org/cmake/help/v3.19/command/set_property.html)

```cmake
set_property(<GLOBAL                      |
              DIRECTORY [<dir>]           |
              TARGET    [<target1> ...]   |
              SOURCE    [<src1> ...]
                        [DIRECTORY <dirs> ...] |
                        [TARGET_DIRECTORY <targets> ...]
              INSTALL   [<file1> ...]     |
              TEST      [<test1> ...]     |
              CACHE     [<entry1> ...]    >
             [APPEND] [APPEND_STRING]
             PROPERTY <name> [<value1> ...])
```

在给定的作用域内设置 0 个或多个对象的属性，第一个参数决定了作用域的范围。

| 作用域    | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| GLOBAL    | 全局唯一                                                     |
| DIRECTORY | 作用域默认为当前路径，也可以使用相对路径或绝对路径指定其它目录。 |
| TARGET    | Scope may name zero or more existing targets. See also the [`set_target_properties()`](https://cmake.org/cmake/help/v3.19/command/set_target_properties.html#command:set_target_properties) command. |
| SOURCE    | Scope may name zero or more source files.                    |
| INSTALL   | Scope may name zero or more installed file paths.            |
| TEST      | Scope may name zero or more existing tests. See also the [`set_tests_properties()`](https://cmake.org/cmake/help/v3.19/command/set_tests_properties.html#command:set_tests_properties) command. |
| CACHE     | Scope must name zero or more cache existing entries.         |

`PROPERTY` 后接属性名，然后是一个值列表。

如果提供了 `APPEND` 选项，则将列表追加到任何现有属性值。

## 五、常见用法

### （一）、设置构建类型

我们的应用程序通常可分为 Release 和 Debug 两种类型，可由变量 `CMAKE_BUILD_TYPE` 指定。

| 类型           | 描述                                          |
| -------------- | --------------------------------------------- |
| Release        | Adds the `-O3 -DNDEBUG` flags to the compiler |
| Debug          | Adds the `-g` flag                            |
| MinSizeRel     | Adds `-Os -DNDEBUG`                           |
| RelWithDebInfo | Adds `-O2 -g -DNDEBUG` flags                  |

- 工程目录结构

  [example-source](https://github.com/ttroy50/cmake-examples/tree/master/01-basic/F-build-type)

  ```bash
  $ tree
  .
  ├── CMakeLists.txt
  ├── main.cpp
  ```

- CMakeLists.txt

  ```
  # Set the minimum version of CMake that can be used
  # To find the cmake version run
  # $ cmake --version
  cmake_minimum_required(VERSION 3.5)
  
  # Set a default build type if none was specified
  if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message("Setting build type to 'RelWithDebInfo' as none was specified.")
    set(CMAKE_BUILD_TYPE RelWithDebInfo CACHE STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
      "MinSizeRel" "RelWithDebInfo")
  endif()
  
  # Set the project name
  project (build_type)
  
  # Add an executable
  add_executable(cmake_examples_build_type main.cpp)
  ```

### （二）、设置编译参数

- 设置变量

  | 变量               | 描述                      |
  | ------------------ | ------------------------- |
  | CMAKE_C_COMPILER   | 变量，可指定 C 编译器     |
  | CMAKE_CXX_COMPILER | 变量，可指定 C++ 编译器   |
  | CMAKE_C_FLAGS      | 变量，可指定 C 编译参数   |
  | CMAKE_CXX_FLAGS    | 变量，可指定 C++ 编译参数 |
  | CMAKE_LINKER       | 变量，指定连接器          |

  例如：

  ```bash
  cmake .. -DCMAKE_C_COMPILER=clang-3.6 -DCMAKE_CXX_COMPILER=clang++-3.6
  ```

- 函数设置

  | 函数                | 描述                                                         |
  | ------------------- | ------------------------------------------------------------ |
  | add_compile_options | add_compile_options(-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes) |
  | ADD_DEFINITIONS     | ADD_DEFINITIONS("-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes) |
  | set                 | set(CMAKE_C_FLAGS "-Wall -Werror -Wstrict-prototypes -Wmissing-prototypes) |

  `add_compile_options` 和 `ADD_DEFINITIONS` 对 C 和 C++ 同时生效，`set` 根据变量名对特定语言生效。

- 工程目录结构

  [example-source](https://github.com/ttroy50/cmake-examples/tree/master/01-basic/G-compile-flags)

  ```bash
  $ tree
  .
  ├── CMakeLists.txt
  ├── main.cpp
  ```

- CMakeLists.txt

  ```cmake
  cmake_minimum_required(VERSION 3.5)
  
  # Set a default C++ compile flag
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)
  
  # Set the project name
  project (compile_flags)
  
  # Add an executable
  add_executable(cmake_examples_compile_flags main.cpp)
  
  target_compile_definitions(cmake_examples_compile_flags 
      PRIVATE EX3
  )
  ```

### （二）、设置构建方式

CMake是跨平台、多语言、多构建方式的构建系统，其不仅支持 `make` 方式，也支持类似 `ninja` 这类性能更好的构建器。

```bash
Generators

The following generators are available on this platform:
  Unix Makefiles              = Generates standard UNIX makefiles.
  Ninja                       = Generates build.ninja files (experimental).
  CodeBlocks - Ninja          = Generates CodeBlocks project files.
  CodeBlocks - Unix Makefiles = Generates CodeBlocks project files.
  Eclipse CDT4 - Ninja        = Generates Eclipse CDT 4.0 project files.
  Eclipse CDT4 - Unix Makefiles
                              = Generates Eclipse CDT 4.0 project files.
  KDevelop3                   = Generates KDevelop 3 project files.
  KDevelop3 - Unix Makefiles  = Generates KDevelop 3 project files.
  Sublime Text 2 - Ninja      = Generates Sublime Text 2 project files.
  Sublime Text 2 - Unix Makefiles
                              = Generates Sublime Text 2 project files.Generators
```



#### 命令行构建器

* Borland Makefiles

* MSYS Makefiles

* MinGW Makefiles

* NMake Makefiles

* NMake Makefiles JOM

* Ninja

* Unix Makefiles

* Watcom WMake

#### IDE 构建器

* Visual Studio 6

* Visual Studio 7

* Visual Studio 7 .NET 2003

* Visual Studio 8 2005

* Visual Studio 9 2008

* Visual Studio 10 2010

* Visual Studio 11 2012

* Visual Studio 12 2013

* Xcode

#### 外部构建器

* CodeBlocks

* CodeLite

* Eclipse CDT4

* KDevelop3

* Kate

* Sublime Text 2



这里以 ninja 为例，首先安装 ninja

```bash
sudo apt-get install ninja-build
```

其次使用命令行选项  `-G` 指定构建方式：

```bash
cmake .. -G Ninja
```

最后构建：

```
ninja -v
```

