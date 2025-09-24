---
layout: post
title: 现代CMake3-变量与缓存
date: 2025-09-08 19:40:18
modify: 2025-09-08 19:40:18
categories: [cmake]
tags: [cmake, cpp]
description: CMake学习笔记-变量与缓存
---


## 现代CMake3-变量与缓存

### 本地变量

声明一个本地 ( local ) 变量：

```cmake
set(MY_VARIABLE "value")
```

- 变量名通常全部用大写，变量值跟在其后
- 可以通过 ${} 来解析一个变量，例如 ${MY_VARIABLE}
- CMake 有作用域的概念，在声明一个变量后，只可以在它的作用域内访问这个变量，例如变量不能在子目录中使用
- 通过在变量声明末尾添加 PARENT_SCOPE 来将它的作用域置定为当前的上一级作用域

列表：

```cmake
set(MY_LIST "one" "two")
set(MY_LIST "one;two")
```

有一些和 `list(` 进行协同的命令， separate_arguments 可以把一个以空格分隔的字符串分割成一个列表。需要注意的是，在 CMake 中如果一个值没有空格，那么加和不加引号的效果是一样的。这使你可以在处理知道不可能含有空格的值时不加引号。

当一个变量用 ${} 括起来的时候，空格的解析规则和上述相同。对于路径来说要特别小心，路径很有可能会包含空格，因此你应该总是将解析变量得到的值用引号括起来，也就是，应该这样 `"${MY_PATH}"`

### 缓存变量

CMake 提供了一个缓存变量来允许你从命令行中设置变量。CMake 中已经有一些预置的变量，像 `CMAKE_BUILD_TYPE` 。如果一个变量还没有被定义，你可以这样声明并设置它。

```cmake
set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "Description")
```

这么写不会覆盖已定义的值。这是为了让你只能在命令行中设置这些变量，而不会在 CMake 文件执行的时候被重新覆盖。如果你想把这些变量作为一个临时的全局变量，你可以这样做：

```cmake
set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "" FORCE)
mark_as_advanced(MY_CACHE_VARIABLE)
```

第一行将会强制设置该变量的值，第二行将使得用户运行 `cmake -L ..` 或使用 GUI 界面的时候不会列出该变量。此外，你也可以通过 `INTERNAL` 这个类型来达到同样的目的（尽管在技术上他会强制使用 `STRING` 类型，这不会产生任何的影响）：

```cmake
set(MY_CACHE_VARIABLE "VALUE" CACHE INTERNAL "")
```

因为 BOOL 类型非常常见，你可以这样非常容易的设置它：

```cmake
option(MY_OPTION "This is settable from the command line" OFF)
```

对于 BOOL 这种数据类型，对于它的 ON 和 OFF 有几种不同的说辞 (wordings) 。

你可以查看 cmake-variables 来查看 CMake 中已知变量的清单。

### 环境变量

通过 `set(ENV{variable_name} value)` 和 `$ENV{variable_name}` 来设置和获取环境变量，不过一般来说，我们最好避免这么用。

### 缓存

缓存实际上就是个文本文件，`CMakeCache.txt` ，当你运行 CMake 构建目录时会创建它。 CMake 可以通过它来记住你设置的所有东西，因此你可以不必在重新运行 CMake 的时候再次列出所有的选项。

### 属性

CMake 也可以通过属性来存储信息。这就像是一个变量，但它被附加到一些其他的物体 ( item ) 上，像是一个目录或者是一个目标。一个全局的属性可以是一个有用的非缓存的全局变量。许多目标属性都是被以 CMAKE_为前缀的变量来初始化的。例如你设置 CMAKE_CXX_STANDARD 这个变量，这意味着你之后创建的所有目标的 CXX_STANDARD 都将被设为CMAKE_CXX_STANDARD 变量的值。

你可以这样来设置属性：

```cmake
set_property(TARGET TargetName
             PROPERTY CXX_STANDARD 11)

set_target_properties(TargetName PROPERTIES
                      CXX_STANDARD 11)
```

第一种方式更加通用 ( general ) ，它可以一次性设置多个目标、文件、或测试，并且有一些非常有用的选项。第二种方式是为一个目标设置多个属性的快捷方式。
