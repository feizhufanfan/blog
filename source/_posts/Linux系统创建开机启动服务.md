---
title: Linux系统创建开机启动服务
date: 2022-02-18 13:37:01
categories:
- linux
- 开机自启服务
tags:
- linux
- 开机自启
- Ubuntu/Debian
- CentOS/UOS
---
# linux下如何创建开机自启服务
## 背景
有时候我们自己写了一些服务程序，需要在开机的时候让他们自动的运行起来。并且有时候还需要根据服务的依赖规定启动顺序。

## 实现
首先我们需要明确我们的系统是什么类型的，因为不同系统设置开机启动的服务是不一样的。
    ```sh
    uname -a
    ```
    ![uname -a](https://gitee.com/feizudefanfan/feizhufanfan_image/raw/master/blog/20220218161322.png)


## 测试
![测试](https://gitee.com/feizudefanfan/feizhufanfan_image/raw/master/blog/20220218161758.png)