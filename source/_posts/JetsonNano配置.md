---
title: JetsonNano配置
date: 2022-02-16 14:21:39
categories:
- linux
- jetson_Nano
tags: 
- jetson Nano
- arm64
- aarch64交叉编译工具链
---


<h1 align="center">Jetson Nano 交叉编译工具链搭建</h1>

## 背景
有时候在Nano上编译程序，使用的IDE可能会有限制亦或者没有显示器只能通过SSH、DRP去链接进行代码编写。如果网络不稳定或者Nano不在身边，那我们只能搭建一个与Nano相同的环境。便可以继续完成工程代码的编写了。
## 工具链下载
1. GCC 7.3.1 for 64 bit BSP and Kernel 交叉编译工具链必须
  https://developer.nvidia.com/embedded/dlc/l4t-gcc-7-3-1-toolchain-64-bit-32-1

2. Sample Root Filesystem Nano简化版的文件系统包包含了工具额外的头文件和库
   https://developer.nvidia.com/embedded/l4t/r32_release_v6.1/t210/tegra_linux_sample-root-filesystem_r32.6.1_aarch64.tbz2

## 解压工具链
解压gcc-chain-tool得到一下文件
![gcc-chain-tool](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219021907.png)

解压文件系统fileSysytem得到以下文件
![fileSysytem](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219015043.png)


## 使用工具链编译项目  
这里推荐使用Cmake来配置项目后生成Makefile文件

使用cmake配置工具链有文件内部配置和GUI界面配置两种方式。
1. 文件配置方式  
    编写CmakeList.txt文件内容如下:
    ```cmake
    cmake_minimum_required(VERSION 3.2)

    #配置目标系统类型
    set(CMAKE_SYSTEM_NAME linux)
    #2 配置C编译器
    set(CMAKE_C_COMPILER /home/du/SoftWave/下载/l4t-gcc-7-3-1-toolchain-64-bit/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc)
    #3 配置C++编译器
    set(CMAKE_CXX_COMPILER   /home/du/SoftWave/下载/l4t-gcc-7-3-1-toolchain-64-bit/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-g++)

    #5 设置目标环境的位置
    SET(CMAKE_FIND_ROOT_PATH /home/du/SoftWave/下载/tegra_linux_sample-root-filesystem_r32.6.1_aarch64/usr/include)


    set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM BOTH)
    set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
    set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

    PROJECT(serial VERSION 1.2.0 LANGUAGES C CXX)
    add_executable(${CMAKE_PROJECT_NAME} main.cpp)
    ```
    执行cmake构建Makefile文件

    ![cmake](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219023707.png)

    编译文件  
    `make`

    查看编译好的可执行文件类型

    ![file](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219023929.png)

    可以看出文件为aarch64的也就是在Nano上运行的执行文件

2. GUI界面配置方式
    编写CmakeList.txt
    ```cmake
    cmake_minimum_required(VERSION 3.2)
    PROJECT(serial VERSION 1.2.0 LANGUAGES C CXX)
    add_executable(${CMAKE_PROJECT_NAME} main.cpp)
    ```
    启动cmake-gui  
    点击Configure配置工程选择最后一个交叉编译配置
    ![Configure](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219024935.png)

    根据目标主机的类型填写系统类型，C/C++/fortran编译器在工具里找到并设置
    ![setting](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219025130.png)

    
    Target Root选择目标主机的运行环境，如果没有使用扩展库可以设定为工具链下的include目录  
    (l4t-gcc-7-3-1-toolchain-64-bit/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/include)

    使用了扩展库则配置下载的文件系统的include目录    
    (tegra_linux_sample-root-filesystem_r32.6.1_aarch64/usr/include)

    其余参数如图所示
    ![](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219030208.png)

    点击finish后在点击Generate生成Makefile文件  
    ![](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219030427.png)
    关掉cmake-gui

    `make`
    
    ![](https://blog.feizhufanfan.top:18088/minio/images/blog/20220219030617.png)

    同样可以看出文件为aarch64的也是在Nano上运行的执行文件
