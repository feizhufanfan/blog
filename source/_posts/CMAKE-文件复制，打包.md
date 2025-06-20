---
title: CMAKE--文件复制，打包
date: 2023-11-17 11:31:37
categories:
- CMAKE
- 文件复制-打包
tags:
- CMAKE
- add_custom_target
---
---

## 文件复制
```
# 文件复制
# 设置在目标编译完成之后将目标复制到指定路径下---注意需要指定路径下存在对应文件夹---
add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD COMMAND ${CMAKE_COMMAND} -E copy_if_different
            $<TARGET_FILE:${PROJECT_NAME}>  # 复制的源文件路径
            ${CMAKE_BINARY_DIR}/${PROJECT_NAME}/test  # 目标路径
            COMMENT "Copying ${PROJECT_NAME} to destination folder"
            )

# CopyTask 用于创建复制任务的名称，实际不输出文件
add_custom_target(CopyTask
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/log.txt ${CMAKE_CURRENT_SOURCE_DIR}/bin/plugin/
		)

```


## 文件打包
```

# 文件打包
# CopyTask 用于创建打包文件任务
add_custom_target(ReleaseTar
		COMMAND ${CMAKE_COMMAND} -E tar "cfvz" "Applications.tar.gz" "${EXECUTABLE_OUTPUT_PATH}"
		)

```

