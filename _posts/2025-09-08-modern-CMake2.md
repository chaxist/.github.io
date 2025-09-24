---
layout: post
title: 现代CMake2-基础知识
date: 2025-09-08 18:46:55
modify: 2025-09-08 18:46:55
categories: [cmake]
tags: [cmake, cpp]
description: CMake学习笔记-基础知识
---


## 现代CMake2-基础知识

### 基础知识

最低版本介绍，必须要包含的第一行。

```cmake
cmake_minimum_required(VERSION 3.16)
```

作者推荐，开始新项目时，可以这么写。但是我对于cmake版本的区别要求不严格，且cmake编译的版本通常比较确定，不会多平台进行编译（这个还是要了解一些的）。

```cmake
cmake_minimum_required(VERSION 3.7...3.21)

if(${CMAKE_VERSION} VERSION_LESS 3.12)
    cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()
```

### 设置一个项目

接下来，每一个顶层 CMakelists 文件都应该加入下面这一行：

```cmake
project(tutorial VERSION 1.0
  DESCRIPTION "A tutorial project"
  LANGUAGES CXX)
```

为了避免缩进问题，我用 vscode 中的 CMake IntelliSence 来帮助我控制缩进。

### 生成一个可执行文件

```cmake
add_executable(one two.cpp three.h)
```

这里有个问题，我在开发中没有往cmake中写过h文件，教程这里却加入了h文件。问了ai，它给出的理由是，IDE体验头文件可见、依赖跟踪更加准确、项目维护结构清晰。小项目或者快速原型验证可不加，中大型项目建议添加**公共头文件**，内部头文件可以不加。

### 生成一个库

```cmake
add_library(one STATIC two.cpp three.h)
```

可以选择库的类型，`STATIC`,`SHARED`或`MODULE`，还有一些虚构的目标（一个不需要编译的目标），例如只有一个头文件的库。这叫做`INTERFACE`库。

### 目标时常伴随着你

现在我们已经指定了一个目标，那我们如何添加关于它的信息呢？例如，它可能需要包含一个目录：

```cmake
target_include_directories(one PUBLIC include)
```

`PUBLIC` 对于一个可执行文件目标没有什么含义；但对于库来说，它让 CMake 知道，任何链接到这个目标的目标也必须包含这个目录。其他选项还有 `PRIVATE`（只影响当前目标，不影响依赖），以及 `INTERFACE`（只影响依赖）。
将目标链接起来：

```cmake
add_library(another STATIC another.cpp another.h)
target_link_libraries(another PUBLIC one)
```

`target_link_libraries`命令，为此目标添加一个依赖 `one`。如果 CMake 项目中不存在名称为 one 的目标（没有定义该 target/目标），那它会直接添加名字为 one 的库到依赖中（一般而言，会去 /usr、CMake 项目指定寻找库的路径等所有能找的路径找到叫 one 的库——译者注）。
