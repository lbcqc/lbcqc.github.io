---

layout: post
title: CMake：安装
categories: [CMake]
description: some word here
keywords: keyword1, keyword2

---

这篇文章记录一下 CMake 在编译之后如何安装程序，并且打包成对应的安装包。

## install

CMake 有内置的函数 `install` 供我们设置工程的安装规则，并生成安装命令 `make install`。

```cmake
install(TARGETS <target>... [...])
install({FILES | PROGRAMS} <file>... [...])
install(DIRECTORY <dir>... [...])
install(SCRIPT <file> [...])
install(CODE <code> [...])
install(EXPORT <export-name> [...])
```

| 类型      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| TARGETS   | 安装 CMake 目标，这个目标由 `add_` 类型命令定义，通常为生成的可执行文件，静态库，动态库 |
| FILES     | 安装普通文件，比如文档、配置文件、脚本工具等                 |
| DIRECTORY | 安装目录，文档、头文件目录、配置文件目录等                   |
| SCRIPT    | 安装时执行某个脚本                                           |
| CODE      | 安装时执行命令，比如 message 打印日志信息                    |
| EXPORT    | 导出 ${PROJECT_NAME}.cmake, 其它工程可以通过find_package找到该项目 |

[详细介绍](https://cmake.org/cmake/help/v3.19/command/install.html)

### （一）基本语法

| 关键字           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| DESTINATION      | 指定目标将要安装的目的目录名，以 `CMAKE_INSTALL_PREFIX` 变量指定的路径为前缀 |
| PERMISSIONS      | 指定目标权限：OWNER_READ, OWNER_WRITE, OWNER_EXECUTE, GROUP_READ, GROUP_WRITE, GROUP_EXECUTE, WORLD_READ, WORLD_WRITE, WORLD_EXECUTE, SETUID, and SETGID |
| CONFIGURATIONS   | 根据构建类型可指定安装分支，构建类型可为：Release, Debug 等  |
| COMPONENT        | 指定安装目标为一个组件，不指定则统一为默认组件 Unspecified，默认安装命令会安装所有组件 |
| EXCLUDE_FROM_ALL | 可指定某个组件不被默认安装                                   |
|                  |                                                              |

### （二）Install Target

```cmake
install(TARGETS targets... [EXPORT <export-name>]
        [[ARCHIVE|LIBRARY|RUNTIME|OBJECTS|FRAMEWORK|BUNDLE|
          PRIVATE_HEADER|PUBLIC_HEADER|RESOURCE]
         [DESTINATION <dir>]
         [PERMISSIONS permissions...]
         [CONFIGURATIONS [Debug|Release|...]]
         [COMPONENT <component>]
         [NAMELINK_COMPONENT <component>]
         [OPTIONAL] [EXCLUDE_FROM_ALL]
         [NAMELINK_ONLY|NAMELINK_SKIP]
        ] [...]
        [INCLUDES DESTINATION [<dir> ...]]
        )
```

### （三）Install Files

```cmake
install(<FILES|PROGRAMS> files...
        TYPE <type> | DESTINATION <dir>
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>]
        [RENAME <name>] [OPTIONAL] [EXCLUDE_FROM_ALL])
```

### （四）Installing Directories

```
install(DIRECTORY dirs...
        TYPE <type> | DESTINATION <dir>
        [FILE_PERMISSIONS permissions...]
        [DIRECTORY_PERMISSIONS permissions...]
        [USE_SOURCE_PERMISSIONS] [OPTIONAL] [MESSAGE_NEVER]
        [CONFIGURATIONS [Debug|Release|...]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL]
        [FILES_MATCHING]
        [[PATTERN <pattern> | REGEX <regex>]
         [EXCLUDE] [PERMISSIONS permissions...]] [...])
```

### （五）Custom Installation Logic

```cmake
install([[SCRIPT <file>] [CODE <code>]]
        [COMPONENT <component>] [EXCLUDE_FROM_ALL] [...])
```

```cmake
install(CODE "MESSAGE(\"Sample install message.\")")
```

### （六）Installing Exports

```cmake
install(EXPORT <export-name> DESTINATION <dir>
        [NAMESPACE <namespace>] [[FILE <name>.cmake]|
        [PERMISSIONS permissions...]
        [CONFIGURATIONS [Debug|Release|...]]
        [EXPORT_LINK_INTERFACE_LIBRARIES]
        [COMPONENT <component>]
        [EXCLUDE_FROM_ALL])
install(EXPORT_ANDROID_MK <export-name> DESTINATION <dir> [...])
```

### （七）Example

[example-source](https://github.com/ttroy50/cmake-examples.git)

- Tree

```bash
HOWARLI-MB0 ॐ  ~/workspace/other/cmake-examples/01-basic/E-installing:(196d14h45m|git@master)
5220 ± tree
├── CMakeLists.txt
├── README.adoc
├── cmake-examples.conf
├── include
│   └── installing
│       └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp

3 directories, 6 files
```

- CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.5)

project(cmake_examples_install)

############################################################
# Create a library
############################################################

#Generate the shared library from the library sources
add_library(cmake_examples_inst SHARED
    src/Hello.cpp
)

target_include_directories(cmake_examples_inst
    PUBLIC 
        ${PROJECT_SOURCE_DIR}/include
)

############################################################
# Create an executable
############################################################

# Add an executable with the above sources
add_executable(cmake_examples_inst_bin
    src/main.cpp
)

# link the new hello_library target with the hello_binary target
target_link_libraries( cmake_examples_inst_bin
    PRIVATE 
        cmake_examples_inst
)

############################################################
# Install
############################################################

# Binaries
install (TARGETS cmake_examples_inst_bin
    DESTINATION bin)

# Library
# Note: may not work on windows
install (TARGETS cmake_examples_inst
    LIBRARY DESTINATION lib)

# Header files
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/ 
    DESTINATION include)

# Config
install (FILES cmake-examples.conf
    DESTINATION etc)
```

- cmake

```cmake
HOWARLI-MB0 ॐ  ~/workspace/other/cmake-examples/01-basic/E-installing/build:(196d14h44m|git@master)
5209 ± cmake .. -DCMAKE_INSTALL_PREFIX="/usr/local/install_example"                                                                                                                                      ⏎
-- The C compiler identification is AppleClang 12.0.0.12000032
-- The CXX compiler identification is AppleClang 12.0.0.12000032
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/lishuai/workspace/other/cmake-examples/01-basic/E-installing/build
```

- make install

```bash
HOWARLI-MB0 ॐ  ~/workspace/other/cmake-examples/01-basic/E-installing/build:(196d14h44m|git@master)
5214 ± sudo make install                                                                                                                                                                                 ⏎
Password:
[ 50%] Built target cmake_examples_inst
[100%] Built target cmake_examples_inst_bin
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/install_example/bin/cmake_examples_inst_bin
-- Installing: /usr/local/install_example/lib/libcmake_examples_inst.dylib
-- Installing: /usr/local/install_example/include
-- Installing: /usr/local/install_example/include/installing
-- Installing: /usr/local/install_example/include/installing/Hello.h
-- Installing: /usr/local/install_example/etc/cmake-examples.conf
```





