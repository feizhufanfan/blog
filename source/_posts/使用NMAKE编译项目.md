---
title: 使用MSBuild编译VS项目
date: 2023-08-21 17:17:03
categories:
- Visual_Studio
- MSBuild
tags:
- MSBuild
---


# 使用MSBuild编译VS项目



## 打开VS兼容命令行
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230821172431.png)

## 切换到项目所在位置
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230821172625.png)
``` bash

cd /d F:\Workspace\C++_Project\ormpp\build

```

## 编译
命令格式：
``` bash
MSBuild MyApp.sln -t:Rebuild  -p:Configuration=Release
#参数说明：
-p:Configuration    指定生成的版本(Debug、Release)
-p:Platform         指定生产的平台(x64、x86)
-t：                指定目标的行为(Rebuild、Clean、build)

```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230821180326.png)

