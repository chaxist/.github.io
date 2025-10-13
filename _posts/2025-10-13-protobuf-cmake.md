---
layout: post
title: ROS使用不同版本的protobuf
date: 2025-10-13 11:00:29
modify: 2025-10-13 11:00:47
author: jst
categories: [cmake]
tags: [cmake]
description: protobuf不同版本编译
---


## 背景

在开发 `gazebo` 镜面反射插件时，需要使用到 `gazebo/gazebo.hh` 头文件，而其中在安装时依赖于 `protobuf3.6.1` ，但是仿真环境下默认安装的 `protobuf` 的版本是3.17.1，版本冲突导致编译失败。因此需要安装不同版本的 `protobuf` ，并在 `cmake` 中指定。

## 编译安装到自定义目录

我使用源码进行安装，在项目的 `3rdparty` 目录下已经下载好了一个 `protobuf-3.6.1`，

1. 编译proto3.6.1到自定义目录

    ```bash
    # 进入protobuf 3.6.1源码目录
    cd protobuf-3.6.1

    # 配置编译，安装到自定义目录
    ./autogen.sh
    ./configure --prefix=/usr/local/protobuf-3.6.1

    # 编译和安装
    make -j$(nproc)
    sudo make install
    ```

2. 在CMake中指定路径

    ```cmake
    # 方法1：设置CMAKE_PREFIX_PATH
    set(CMAKE_PREFIX_PATH "/usr/local/protobuf-3.6.1")
    find_package(Protobuf REQUIRED)

    # 方法2：直接指定protobuf路径
    set(Protobuf_ROOT "/usr/local/protobuf-3.6.1")
    find_package(Protobuf REQUIRED)

    # 方法3：手动设置变量
    set(Protobuf_INCLUDE_DIRS "/usr/local/protobuf-3.6.1/include")
    set(Protobuf_LIBRARIES "/usr/local/protobuf-3.6.1/lib/libprotobuf.so")
    set(Protobuf_PROTOC_EXECUTABLE "/usr/local/protobuf-3.6.1/bin/protoc")
    ```

3. 验证版本

    ```bash
    # 检查protoc版本
    /usr/local/protobuf-3.6.1/bin/protoc --version

    # 检查链接的库
    ldd your_executable | grep protobuf
    ```

这样就可以成功编译和使用protobuf 3.6.1版本，同时保持系统中的3.17版本不受影响。

## 遇到问题 fPIC

安装后编译ROS包链接时报错：

```txt
/usr/bin/ld: /usr/local/protobuf-3.6.1/lib/libprotobuf.a(arena.o): relocation R_X86_64_TPOFF32 against symbol _ZN6google8protobuf8internal9ArenaImpl13thread_cache_E' can not be used when making a shared object; recompile with -fPIC /usr/bin/ld: /usr/local/protobuf-3.6.1/lib/libprotobuf.a(common.o): relocation R_X86_64_PC32 against symbol stderr@@GLIBC_2.2.5' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: final link failed: bad value
collect2: error: ld returned 1 exit status
```

这个错误是因为protobuf 3.6.1编译时没有启用位置无关代码(PIC)，而你在链接共享库时需要PIC支持。

重新编译protobuf 3.6.1启用PIC:

1. 清理之前的安装

    ```bash
    cd protobuf-3.6.1
    sudo make uninstall  # 如果之前安装过
    make clean
    ```

2. 重新配置并编译

    ```bash
    # 重新配置，启用PIC和共享库
    ./autogen.sh
    ./configure --prefix=/usr/local/protobuf-3.6.1 \
                --enable-shared \
                --enable-static \
                CXXFLAGS="-fPIC" \
                CFLAGS="-fPIC"
    # 编译和安装
    make -j$(nproc)
    sudo make install
    ```

3. 更新动态库缓存

    ```bash
    echo "/usr/local/protobuf-3.6.1/lib" | sudo tee /etc/ld.so.conf.d/protobuf-3.6.1.conf
    sudo ldconfig
    ```

增加了 `-fPIC` 后，会在编译过程中看到 `libtool: compile:  g++ ... -fPIC` ，那么这就是正确的。现在回到ROS路径下编译包就是正常的了。
