---
title: 关于lib库中rpath、runtime路径问题
date: 2023-03-24 14:52:59
categories:
- CMAKE
- library
tags:
- linux
---



## RPATH问题  

在linux环境中有时候有些so库在运行时依赖于其他第三方库，但在windows下系统会优先查找`System32`、`System`等系统目录。
在这些目录下都没有相关依赖库时会去扫描当前`DLL`运行目录下是包含依赖的`DLL`,而在linux环境中系统优先扫描`LD_LIBRARY_PATH`、`/etc/ld.so.conf`等路径
在这些路径下都没有最后会去执行生产SO库时的`INSTALL_RPATH`路径。


## 解决因路径问题导致第三方依赖库在同级目录下无法加载的问题
    在CMake中设置生产SO/DLL库时，运行路径或者编译安装路径。
    ``` CMakeLists
    set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "-Wl,-rpath,$ORIGIN:./:.")
    set(CMAKE_SKIP_RPATH FALSE)
    set(CMAKE_SKIP_BUILD_RPATH TRUE)
    set(CMAKE_SKIP_INSTALL_RPATH TRUE)
    ```  
    注释：$ORIGIN 为编译系统当中的环境变量，能够设置当前运行的库路径的相对路径值。

