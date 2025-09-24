---
title: 现代CMake4-用CMake进行编程
date: 2025-09-09 19:21:53
modify: 2025-09-09 19:21:53
categories: [cmake]
tags: [cmake, cpp]
description: CMake学习笔记-用CMake进行编程
---


## 现代CMake4-用CMake进行编程

### 流程控制

这是一个 if 语句的例子：

```cmake
if(variable)
    # If variable is `ON`, `YES`, `TRUE`, `Y`, or non zero number
else()
    # If variable is `0`, `OFF`, `NO`, `FALSE`, `N`, `IGNORE`, `NOTFOUND`, `""`, or ends in `-NOTFOUND`
endif()
# If variable does not expand to one of the above, CMake will expand it then try again
```

一些关键字：

- 一元的: `NOT`, `TARGET`, `EXISTS` (文件), `DEFINED`, 等；
- 二元的: `STREQUAL`, `AND`, `OR`, `MATCHES` ( 正则表达式 ), `VERSION_LESS`, `VERSION_LESS_EQUAL` ( CMake 3.7+ ), 等；
- 括号可以用来分组；

### generator-expressions

详细了解去[官网](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html)

你想要他们在构建或者安装的时候运行呢，应该怎么写？ 生成器表达式就是为此而生。
这一部分结合案例来看
> 你几乎会在所有支持安装的软件包中看到如下代码：

  ```c
  target_include_directories(
      MyTarget
    PUBLIC
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
      $<INSTALL_INTERFACE:include>
  )
  ```

### 宏定义与函数

function 或 macro。函数和宏只有作用域上存在区别，宏没有作用域的限制。所以说，如果你想让函数中定义的变量对外部可见，你需要使用 PARENT_SCOPE 来改变其作用域。如果是在嵌套函数中，这会变得异常繁琐，因为你必须在想要变量对外的可见的所有函数中添加 PARENT_SCOPE 标志。但是这样也有好处，函数不会像宏那样对外“泄漏”所有的变量。接下来用函数举一个例子：

```c
function(SIMPLE REQUIRED_ARG)
    message(STATUS "Simple arguments: ${REQUIRED_ARG}, followed by ${ARGN}")
    set(${REQUIRED_ARG} "From SIMPLE" PARENT_SCOPE)
endfunction()

simple(This Foo Bar)
message("Output: ${This}")
```

输出如下：

```c
-- Simple arguments: This, followed by Foo;Bar
Output: From SIMPLE
```

这里有疑惑，`This`是什么，为什么和 Foo;Bar 不会一起打印出来。
**函数定义分析**：这里定义了一个名为 `SIMPLE` 的函数，它有一个必需参数 `REQUIRED_ARG`
**函数调用分析**：当调用函数时，参数分配如下：

- `This` → 赋值给 `REQUIRED_ARG`
- `Foo Bar` → 成为额外参数，存储在特殊变量 `ARGN` 中

> 如果你想要有一个指定的参数，你应该在列表中明确的列出，除此之外的所有参数都会被存储在 ARGN 这个变量中（ ARGV 中存储了所有的变量，包括你明确列出的 ）。CMake 的函数没有返回值，你可以通过设定变量值的形式来达到同样地目的。在上面的例子中，你可以通过指定变量名来设置一个变量的值

### 参数的控制

> 这个东西看懂了但是感觉没有声明用的地方。

CMake 拥有一个变量命名系统。你可以通过 cmake_parse_arguments 函数来对变量进行命名与解析。如果你想在低于 3.5 版本的CMake 系统中使用它，你应该包含 CMakeParseArguments 模块，此函数在 CMake 3.5 之前一直存在与上述模块中。这是使用它的一个例子：

```c
function(COMPLEX)
    cmake_parse_arguments(
        COMPLEX_PREFIX
        "SINGLE;ANOTHER"
        "ONE_VALUE;ALSO_ONE_VALUE"
        "MULTI_VALUES"
        ${ARGN}
    )
endfunction()

complex(SINGLE ONE_VALUE value MULTI_VALUES some other values)
```

在调用这个函数后，会生成以下变量：

```c
COMPLEX_PREFIX_SINGLE = TRUE
COMPLEX_PREFIX_ANOTHER = FALSE
COMPLEX_PREFIX_ONE_VALUE = "value"
COMPLEX_PREFIX_ALSO_ONE_VALUE = <UNDEFINED>
COMPLEX_PREFIX_MULTI_VALUES = "some;other;values"
```

## 与代码交互

### 通过 CMake 配置文件

CMake 允许你在代码中使用 `configure_file` 来访问 `CMake` 变量。该命令将一个文件（ 一般以 `.in` 结尾 ）的内容复制到另一个文件中，并替换其中它找到的所有 CMake 变量。如果你想要在你的输入文件中避免替换掉使用 `${}` 包含的内容，你可以使用 `@ONLY` 关键字。还有一个关键字 `COPY_ONLY` 可以用来作为 `file(COPY` 的替代字。

> [NOTE] 这个功能在 `CMake` 中使用的相当频繁，例如在下面的 `Version.h.in` 中:

Version.h.in

```c
#pragma once

#define MY_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define MY_VERSION_MINOR @PROJECT_VERSION_MINOR@
#define MY_VERSION_PATCH @PROJECT_VERSION_PATCH@
#define MY_VERSION_TWEAK @PROJECT_VERSION_TWEAK@
#define MY_VERSION "@PROJECT_VERSION@"
```

CMake lines:

```c
configure_file (
    "${PROJECT_SOURCE_DIR}/include/My/Version.h.in"
    "${PROJECT_BINARY_DIR}/include/My/Version.h"
)
```

使用 `cmake` 后，会在 build/include/My 文件夹中生成一个 Version.h 文件，长这个样子：

```c
#pragma once

#define MY_VERSION_MAJOR 1
#define MY_VERSION_MINOR 0
#define MY_VERSION_PATCH 
#define MY_VERSION_TWEAK 
#define MY_VERSION "1.0"
```

也就是说，将 `@@` 中的值替换为了cmake中的配置。我在修改一下cmake中的值：

```c
# CMakeLists.txt
set(PROJECT_VERSION_PATCH 2)
set(PROJECT_BUILD_RELEASE True)
set(PROJECT_VERSION_TWEAK "Bold")

# Version.h.in中
#define MY_BUILD_RELEASE @PROJECT_BUILD_RELEASE@
```

得到：

```c
#pragma once

#define MY_VERSION_MAJOR 1
#define MY_VERSION_MINOR 0
#define MY_VERSION_PATCH 2
#define MY_VERSION_TWEAK Bold
#define MY_BUILD_RELEASE True
#define MY_VERSION "1.0"
```

你也可以使用（ 并且是常用 ）这个来生成 .cmake 文件，例如配置文件（ 见 [installing](https://cliutils.gitlab.io/modern-cmake/chapters/install/installing.html) ）。

### 读入文件

另外一个方向也是行得通的， 你也可以从源文件中读取一些东西（ 例如版本号 ）。例如，你有一个仅包含头文件的库，你想要其在无论有无 `CMake` 的情况下都可以使用，上述方式将是你处理版本的最优方案。可以像下面这么写：

```c
# Assuming the canonical version is listed in a single line
# This would be in several parts if picking up from MAJOR, MINOR, etc.
set(VERSION_REGEX "#define MY_VERSION[ \t]+\"(.+)\"")

# Read in the line containing the version
file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/include/My/Version.hpp"
    VERSION_STRING REGEX ${VERSION_REGEX})

# Pick out just the version
string(REGEX REPLACE ${VERSION_REGEX} "\\1" VERSION_STRING "${VERSION_STRING}")

# Automatically getting PROJECT_VERSION_MAJOR, My_VERSION_MAJOR, etc.
project(My LANGUAGES CXX VERSION ${VERSION_STRING})
```

比如创建一个头文件：

```bash
echo "#define MY_VERSION \"1.0.0\"" > ../include/My/Version.hpp
```

就可以读取到 `${VERSION_STRING}` 里面，结果为 `1.0.0` ，需要注意，头文件中要加引号。
