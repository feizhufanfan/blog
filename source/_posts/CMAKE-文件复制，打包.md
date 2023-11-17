---
title: CMAKE--文件复制，打包
date: 2023-11-17 11:31:37
categories:
- CMAKE
- 文件复制、打包
tags:
- CMAKE
- add_custom_target
---
---

## 文件复制
```
# 文件复制
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

