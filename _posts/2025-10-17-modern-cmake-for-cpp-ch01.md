---
layout: post
title: modern-cmake-for-cpp-ch01
date: 2025-10-17 16:22:49
modify: 2025-10-17 16:22:49
author: jst
categories: [cmake]
tags: [cmake]
description:
math: true
---

开个新坑，看看 `modern cmake for cpp` 书，书比较长，作为一个工具，400多页确实太长了。但是想要深入的理解构建代码，可以先看看怎么样，有收获再继续看下去。

## 0.How does it work?

cmake 分为三个阶段：This process has three stages，

- Configuration
- Generation
- Building

## 1. Mastering the command line

主要有五种执行程序：

- cmake：主要的配置、生成和构建的工具
- ctest
- cpack
- cmake-gui
- ccmake

### 1.1 CMake command line

提供一些运行模式：

- Generating a project buildsystem
- Building a project
- Installing a project
- Running a script
- Running a command-line tool
- Running a workflow preset
- Getting help

`cmake` 命令不只是开始编译，还是创建构建系统、构建、安装、运行脚本等等。

#### 1.1.1 Generating a project buildsystem

老外的术语是buildsystem，构建系统，很直观的介绍了cmake就是创建一个编译系统的。

```bash
cmake [<options>] -S <source tree> -B <build tree>
cmake [<options>] <source tree>
cmake [<options>] <build tree>
```

一般使用第一个就行了。区分 `-S` 和 `-B` ，是为了将编译产物和源码分离，便于版本控制。例如：

```bash
cmake -S ./project -B ./build
```

读取project路径， 在build路径下创建构建系统。

#### 1.1.2 Choosing a generator

generator 是 cmake 中的构建系统生成器。使用命令来指定：

```bash
cmake -G <generator name> -S <source tree> -B <build tree>
```

可以在 help 信息中找到自己当前有哪些 generator :

```bash
cmake --help
```

#### 1.1.3 Managing the project cache

编译缓存，配置阶段会收集信息并且缓存在 `CMakeCache.txt` 文件中。

```bash
cmake -D <var>[:<type>]=<value>
```

有一个很重要的参数用来指定构建类型 `CMAKE_BUILD_TYPE`，例如：

```bash
cmake -S . -B ../build -D CMAKE_BUILD_TYPE=Release
```

#### 1.1.4 Debugging and tracing

`cmake` 命令提供选项查看内部参数

```bash
cmake --system-information [file]
```

项目中我们会使用 `message()` 命令来打印构建过程中的细节，`Cmake`命令可以报告这些值：

```bash
cmake --log-level=<level>
```

level 可以使用  ERROR, WARNING, NOTICE, STATUS, VERBOSE, DEBUG, or TRACE，在函数 `message()` 中定义的那个等级，就会打印出来的那个等级的内容

#### 1.1.5 Configuring presets

### 1.2 Building a project

构建项目:

```bash
cmake --build <build tree>
```

多线程构建：

```bash
cmake --build <build tree> --parallel [<number of jobs>]
cmake --build <build tree> -j [<number of jobs>]

```

选择构建目标：每个项目有一个或者多个部分组成，称为 `targets`，编译的时候也可以选择编译

```bash
cmake --build <build tree> --target <target1> --target <target2>
```

清除构建树，`clean` 作为一个特殊的构建对象，构建 `clean` 时将会清除构建目录下的所有产物。

```bash
cmake --build <build tree> -t clean
```

配置编译选项

```bash
cmake --build <build tree> --config <cfg>
```

## 1.3 Install a project

构建后用户需要安装到系统中，意味着拷贝文件到正确的路径、安装库，或者运行用户安装逻辑。

```bash
cmake --install <build tree> [<option>]
```

选择安装路径：

```bash
cmake --install <build tree> --install-prefix
```

## 1.4 Running a script

```bash
cmake -E <command> [<option>]
```




## 2. 导航项目路径和文件

source tree：源码路径，要求 `CMakeLists.txt` 配置文件
build tree: 编译产物，二进制文件、可执行文件、库等，

## 2.1 Listfiles

CMake通过 `include()` 和 `find_package()` 或者 `add_subdirectory()` 添加文件列表。

## 2.2 JSON and YAML files

用来提供简单的配置。