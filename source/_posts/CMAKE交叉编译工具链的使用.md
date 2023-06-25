---
title: CMAKE交叉编译工具链的使用
date: 2023-06-05 11:03:31
categories:
- CMAKE
- TOOLCHAIN
tags:
- CMAKE
- TOOLCHAIN
---


# 交叉编译

## 交叉编译方式
- 指定编译器

```sh

set(CMAKE_SYSTEM_NAME Linux)   #指定运行的目标平台类型 Linux、 Windows、Darwin(macOS)、android
set(CMAKE_SYSTEM_PROCESSOR arm) #指定目标平台的架构

set(CMAKE_SYSROOT /home/devel/rasp-pi-rootfs) #设定目标系统的文件系统
set(CMAKE_STAGING_PREFIX /home/devel/stage)  #目标主机的安装位置   可选参数

set(tools /home/devel/gcc-4.7-linaro-rpi-gnueabihf)
set(CMAKE_C_COMPILER ${tools}/bin/arm-linux-gnueabihf-gcc)  #指定gcc编译器
set(CMAKE_CXX_COMPILER ${tools}/bin/arm-linux-gnueabihf-g++)  #指定g++编译器

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER) #NEVER：使用宿主的编译工具，ONLY：只使用CMAKE_SYSROOT指定目录下的编译工具
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)     
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)

```

- 指定编译工具链
```sh 


set(CMAKE_SYSTEM_NAME Android)  #指定运行的目标平台类型 Linux、 Windows、Darwin(macOS)、android

set(CMAKE_ANDROID_API 21)  #如果是安卓设定SDK版本等级
set(CMAKE_ANDROID_ARCH_ABI aarch64) #安卓设定架构版本
set(CMAKE_ANDROID_STL_TYPE gnustl_static)
set(CMAKE_TOOLCHAIN_FILE )

```

