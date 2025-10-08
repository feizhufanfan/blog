---
title: STM32--qemu模拟仿真STM32开发板
date: 2023-07-17 23:20:50
categories:
- STM32
- Qemu仿真STM32
tags:
- STM32
- Qemu
---

<h1 align="center">使用Qemu仿真STM32F103C8T6</h1>
前言：
有时候对于一些简单的代码逻辑，需要进行修改但刚好身边没有开发板和下载器。那此时我们可用使用Qemu来模拟仿真开发板进行基础功能的验证。

# 环境准备
1. [xPack QEMU Arm v7.2.0-1](https://github.com/xpack-dev-tools/qemu-arm-xpack/releases/download/v7.2.0-1/xpack-qemu-arm-7.2.0-1-win32-x64.zip)

## 环境配置
将`QEMU Arm v7.2.0-1`下载解压到自定义目录并在系统的环境变量`Path`中添加路径。
在cmd窗口中运行`qemu-system-gnuarmeclipse`,出现一下信息说明配置成功。
```
F:\Program Files (x86)\xpack-qemu-arm-7.2.0-1\bin\qemu-system-gnuarmeclipse.exe:
Error: Neither board nor mcu specified, and there is no default.
Use -board help or -mcu help to list supported boards or devices!

```

# 在Clion中配置相关参数
在Clion中的`Debug Configuration`中添加添加`Remote Debug`
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718000701.png)
如图填入相应参数：
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718001025.png)
其中`Target Remote args`固定填写为`127.0.0.1:1234`


# 调试和运行
在编译文件生成位置打开cmd窗口
输入命令`qemu-system-gnuarmeclipse -S -s  -M BluePill --image TIM_demo.elf`,会弹出一个STM32F103C8T6的窗口。此时该窗口中的开发板进入调试等待模式。
命令参数说明：
- -S Debug模式
- -s 冻结CPU，也就是等待调试器连接后才会开始运行
- -M 选择开发板类型
- --images 等待调试的程序 
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718002217.png)

接下来在CLion中选择我们刚刚创建的`Remote Debug`,然后选择点击`debug`按钮进入调试模式
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718002823.png)

在主函数打上断点。
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718003506.png)
可以看出已经成功进入断点。

此时还未复位GPIOC13，LED为灭
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718004159.png)
执行GPIOC13复位后，LED为亮
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718004343.png)
再次执行复位操作后，LED为灭
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230718004419.png)

由此可以判断出代码的逻辑。




