---
title: jetson NanoPWM配置
date: 2022-02-19 23:27:57
categories:
- linux
- jetson Nano
tags:
- PWM
- 番茄计划
---

# 前言
在完成番茄计划中，完成了对Uart的配置后。进行了下一步计划的开展。

# 背景
jetson Nano镜像安装完成之后，准备实现自己的番茄计划。在项目实现过程当中遇到一些需要关注的点，于是便将他们单独罗列出来记录一下。

# 资源查找
原本我以为jetson Nano想要输出PWM波就像树莓派使用`Pi.GPIO`库一样只要配置好引脚然后设置频率和占空比就可以舒舒服服的使用PWM了。  
结果我去官网查找资料的时候看到这么一段话  
这里贴上原文的[链接](https://github.com/NVIDIA/jetson-gpio/blob/master/README.md)  
The Jetson.GPIO library supports PWM only on pins with attached hardware PWM controllers. Unlike the RPi.GPIO library, the Jetson.GPIO library does not implement Software emulated PWM  
这句话就是说jetson Nano有一个PWM硬件控制器支持2通道输出，并且不支持RPI.GPIO的软件模拟输出。  
什么居然直接支持硬件层面的支持，哇塞~ 狂喜，但只有两路还不支持软件模拟有点可惜。

在看看后面  
The system pinmux must be configured to connect the hardware PWM controlller(s) to the relevant pins. If the pinmux is not configured, PWM signals will not reach the pins! The Jetson.GPIO library does not dynamically modify the pinmux configuration to achieve this. Read the L4T documentation for details on how to configure the pinmux.
大概意思就是需要把PWM控制器的输出连接到对应的引脚，否则PWM 信号将无法到达引脚。 这感觉就像是STM32的引脚复用呀。

那么就下来就去看看引脚复用的配置。

## PWM的输出配置
1. 启动Jetson-IO配置工具
   ```sh
   sudo /opt/nvidia/jetson-io/jetson-io.py
   ```  
   ![tools](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220221001829.png)

   回车进入引脚配置  
   ![UI1](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220221002034.png)
   有时候可能会显示不全，把终端窗口拉长一点就会显示出来了  
   ![UI2](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220221002141.png)  
   这种情况也是我意外发现的，2333

   可以图中看到`32\33`引脚是作为PWM的输出引脚。  
   接下来使用方向键选择配置配置引脚
   ![UI3](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220221002523.png)  
   回车进入界面进行配置，使用空格键进行选择.由于我这里已经选好了就`[]`里带有`*`的标识。  
   ![UI4](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220221002619.png)
   选好后返回上一界面选择`Save pin changes`  
   ![UI5](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220221003202.png) 
   再选择`Save and exit without rebooting`这个选项。
   ![UI6](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220221003336.png)  
   回车确认，即可完成PWM的输出引脚配置啦












参考资料:
1. [Configuring Jetson Expansion Header](https://docs.nvidia.com/jetson/l4t/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/hw_setup_jetson_io.html)