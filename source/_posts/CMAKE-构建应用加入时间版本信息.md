---
title: CMAKE--构建应用加入时间版本信息
date: 2023-11-17 11:32:15
categories:
- CMAKE
- 获取时间、git版本信息
tags:
- CMAKE
- add_custom_target
---
---



## 获取系统时间
```

string(TIMESTAMP COMPILE_TIME %Y%m%d_%H%M%S)


```

## 配置版本信息模板

```

configure_file(
		"${CMAKE_SOURCE_DIR}/VersionConfig.h.in"
		"${CMAKE_SOURCE_DIR}/VersionConfig.h"
)

```

``` C++ VersionConfig.h.in

//VersionConfig.h.in
#define V_BUILD_TIME "@build_time@"
#define V_GIT_INFO "@GIT_BRANCH@_@GIT_HASH@"

```
