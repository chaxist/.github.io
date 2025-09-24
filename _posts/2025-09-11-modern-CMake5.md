---
layout: post
title: 现代CMake5-组织你的项目
date: 2025-09-11 19:16:35
modify: 2025-09-11 19:16:35
categories: [cmake]
tags: [cmake, cpp]
description: CMake学习笔记-组织你的项目
---


## 现代CMake5-组织你的项目

首先，如果你创建一个名为 project 的项目，它有一个名为 lib 的库，有一个名为 app 的可执行文件，那么目录结构应该如下所示：

```txt
- project
  - .gitignore
  - README.md
  - LICENCE.md
  - CMakeLists.txt
  - cmake
    - FindSomeLib.cmake
    - something_else.cmake
  - include
    - project
      - lib.hpp
  - src
    - CMakeLists.txt
    - lib.cpp
  - apps
    - CMakeLists.txt
    - app.cpp
  - tests
    - CMakeLists.txt
    - testlib.cpp
  - docs
    - CMakeLists.txt
  - extern
    - googletest
  - scripts
    - helper.py
```

1. `include` 中没有 `CMakeLists.txt`文件，为了将其拷贝到 `/usr/include` 或者其他类似目录下，避免冲突问题，就不能有除了头文件以外的文件。

2. 需要有一个 `cmake` 文件夹，里面包含所有用到的辅助模块。这是你放置所有 Find*.cmake 的文件。你可以在 [github.com/CLIUtils/cmake](https://github.com/CLIUtils/cmake) 找到一些常见的辅助模块集合。你可以通过以下语句将此目录添加到你的`CMake Path` 中：

    ```c
    set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
    ```

3. `extern` 几乎只包含git子模块。

[扩展代码详例](https://github.com/Modern-CMake-CN/Modern-CMake-zh_CN/tree/master/examples/extended-project)

命令比较厉害：

```bash
cmake -S . -B build
cmake --build build
# To test (--target can be written as -t in CMake 3.15+):
cmake --build build --target test
# To build docs (requires Doxygen, output in build/docs/html):
cmake --build build --target docs
```

我们项目底下只有一个 `cartographer` 的 `index/html` 。

## 在CMake中运行其他程序

### 在配置时运行一条命令

可以在脚本模式下尝试，`cmake -P xxx.cmake`。

可以使用 execute_process 来运行一条命令并获得他的结果。一般来说，在 `CMake` 中避免使用硬编码路径是一个好的习惯，你也可以使用 `${CMAKE_COMMAND}` ,`find_package(Git)` , 或者`find_program` 来获取命令的运行权限。可以使用 `RESULT_VARIABLE` 变量来检查返回值，使用 `OUTPUT_VARIABLE` 来获得命令的输出。

书中的例子，有点麻烦，我没有子模块的 `git` 好像并没有执行。

```c
find_package(Git QUIET)

if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                    RESULT_VARIABLE GIT_SUBMOD_RESULT)
    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
        message(FATAL_ERROR "git submodule update --init --recursive failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
    endif()
endif()
```

几个比较重要的参数：

```c
execute_process(COMMAND <cmd1> [<arguments>]
                [COMMAND <cmd2> [<arguments>]]...
                [WORKING_DIRECTORY <directory>]
                [TIMEOUT <seconds>]
                [RESULT_VARIABLE <variable>]
                [RESULTS_VARIABLE <variable>]
                [OUTPUT_VARIABLE <variable>]
                [ERROR_VARIABLE <variable>]
                [INPUT_FILE <file>]
                [OUTPUT_FILE <file>]
                [ERROR_FILE <file>]
                [OUTPUT_QUIET]
                [ERROR_QUIET]
                [COMMAND_ECHO <where>]
                [OUTPUT_STRIP_TRAILING_WHITESPACE]
                [ERROR_STRIP_TRAILING_WHITESPACE]
                [ENCODING <name>]
                [ECHO_OUTPUT_VARIABLE]
                [ECHO_ERROR_VARIABLE]
                [COMMAND_ERROR_IS_FATAL <ANY|LAST|NONE>])
```

- WORKING_DIRECTORY :在指定目录执行
- TIMEOUT：子进程超时限制
- RESULT_VARIABLE：接受上一个子进程的结果。它有两种形式：整数，表达错误码；字符串，表达一个错误条件
- OUTPUT_VARIABLE： 接受子进程的标准输出（stdout）
- ERROR_VARIABLE： 接受子进程的标准错误输出（stderr）

例如：

```c
execute_process(
    COMMAND echo "world"
    RESULT_VARIABLE ECHO_RESULT
    OUTPUT_VARIABLE ECHO_OUTPUT
)

if(ECHO_RESULT EQUAL "0")
    message(STATUS "${ECHO_OUTPUT}")
endif()
```

### 在构建时运行一条命令

### CMake 中包含的常用的工具

一个有用的工具是 `cmake -E <mode>`（在 `CMakeLists.txt` 中被写作 `${CMAKE_COMMAND} -E`）。通过指定后面的 `<mode>` 允许 CMake 在不显式调用系统工具的情况下完成一系列事情，例如 `copy`(复制)`，make_directory`(创建文件夹)，和 `remove`(移除) 。
