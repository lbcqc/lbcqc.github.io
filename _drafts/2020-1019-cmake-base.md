---
layout: post
title: CMake：基础
categories: [CMake]
description: some word here
keywords: keyword1, keyword2
---

# 概述

工作中多处使用到CMAKE，以前学习到Makefile，借此机会，系统的学习整理下CMAKE。  

## 简介

- wiki：<https://en.wikipedia.org/wiki/CMake>
- 官方文档：<https://cmake.org/documentation/>
- cmake-example：<https://github.com/ttroy50/cmake-examples.git>

CMake是一个管理构建过程的开源软件，支持多平台的、多编译器、多构建系统。  

- 平台：Linux，MaxOS，Windows，Cygwin
- 编译器：Gcc(默认)，Clang
- 构建系统：Makefile(默认)，ninja，Qt Creator，Xcode，Microsoft Visual Studio

1. CMake和Makefile的区别？

    CMake可以使用Makefile作为构建工具来管理构建过程，同时CMake还支持其它构建系统，比如ninja。CMake相比于Makefile更容易处理跨平台问题，更容易管理第三方库、导入包等等。

2. CMake和Automake的比较？

    两者都是通过Makefile等构建系统工作，都是跨平台的，但是CMake出现的更晚，功能更加丰富，构建过程更高效。

## HelloWorld

最简单的情况下，没有头文件，没有第三方库，只需要将所有的源文件编译链接生成一个可执行文件，如下所示：

```bash
.
├── CMakeLists.txt
├── main.cpp
```

```CMake
# 设置cmake最小版本
cmake_minimum_required(VERSION 3.5)

# 工程名（注意，test已经被CMake测试组件CTest当作预留工程名）
project (hello_cmake)

# 添加可执行文件
add_executable(hello_cmake main.cpp)
```

## 