---
title: CMAKE-指定程序生成位置
date: 2023-05-14 19:38:44
categories:
- CMAKE
- set_target_properties
tags:
- CMAKE
---

# 关于CMAKE在生成可执行文件的位置设定

## 使用`set_target_properties`
``` sh
set_target_properties(${PROJECT_NAME} #目标名称
        PROPERTIES
        LINKER_LANGUAGE CXX #链接器语言
        ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR} 
        LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}
        RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}
        )
```

## 注意：
在Windows下时使用MSVC编译生成目标位置时会自动添加`${CMAKE_BUILD_TYPE}`使得输出位置与其他编译器有所差异，因此需要对编程类型做出判断
如下:
``` sh
if(${CMAKE_GENERATOR} MATCHES "Visual Studio*")
    #在Visual Studio生成器（即VS工程）下，会在EXECUTABLE_OUTPUT_PATH、EXECUTABLE_OUTPUT_PATH后面自动加一个${CMAKE_BUILD_TYPE}
    set_target_properties(${PROJECT_NAME}
            PROPERTIES
            LINKER_LANGUAGE CXX
            ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}
            LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}
            RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}
     )
else()
    set_target_properties(${PROJECT_NAME}
            PROPERTIES
            LINKER_LANGUAGE CXX
            ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/${CMAKE_BUILD_TYPE}
            LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/${CMAKE_BUILD_TYPE}
            RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/${CMAKE_BUILD_TYPE}
    )

endif()
```
