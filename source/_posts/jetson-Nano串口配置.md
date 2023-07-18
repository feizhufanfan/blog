---
title: jetson Nano串口配置
date: 2022-02-19 17:39:48
categories:
- Linux
- jetson Nano
tags:
- UART
- 番茄计划
---
# 背景
jetson Nano镜像安装完成之后，准备实现自己的番茄计划。在项目实现过程当中遇到一些需要关注的点，于是便将他们单独罗列出来记录一下。

## UART的配置
由于Jetson Nano的串口引脚布局和树莓派的布局几乎一致（我想应该时树莓派在极客圈很火相关的资源也比较多。为了很好的融合这个圈子保持引脚标号上将树莓派作为参照，既能够使用树莓派相关的资源也方便极客们更快的上手Nano),查询很多资源后得知串口的通信基本上都是用Pyhton来进行操作的。作为一名C++为主打的码农怎么能只会Python调库,必须C++来实现串口的读写。

首先要理解串口在linux系统中的存在形式。按照linux的哲学一切都是文件。也就是说串口这个设备多Linux来说就是一个文件。当我们需要使用串口进行通信时，只需要按照串口通信的配置配置该文件，然后对文件进行读写操作即可实现串口收发。理解了这一形式那开始验证。


1. 首先接线  
   
   有图中红框标出的部分，可以看出串口的引脚为8、9设备名称为`ttyTHS1`
   ![Uart-pin](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219184923.png)

   这里我使用的时PL2303USB-TTL作为与Nano通信的工具。

   未插上USB-TTL时,设备列表如图  
   ![设备未插](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219190211.png)

   插上设备后可以看到新增加了一个`ttyUSB0`的串口设备  
   ![插上设备](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219190317.png)  

   然后将USB-TTL的RX，TX接到Nano的8、10引脚  
   ![jetson Nano](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220226210741.png)  
  

2. 对串口设备（文件）赋予读写权限
   主机上  
   `sudo chmod a+wr /dev/ttyUSB0`
   ![主机](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219192132.png) 
   Nano上  
   `sudo chmod a+wr /dev/ttyTHS1`  
   ![Nano](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219192332.png)

3. 对串口进行配置
   主机上  
   `sudo stty -F /dev/ttyUSB0 speed 115200 cs8 -parenb -cstopb` 
   设置串口`ttyUSB0`波特率为115200，8位数据位，1位停止位，无校验位
   ![设置波特率](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219225640.png)  
   第一次设置的时候有一个默认波特率，在进行一次设置确保为设置的数值。
   Nano上  
   ![设置波特率](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219230105.png)

4. 通信测试
   主机发给Nano
   主机往`/dev/ttyUSB0`写入要发送的数据，Nano读取`/dev/ttyTHS1`里的数据。
   ![主机发给Nano](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219231021.png)  
   可以看到Nano读取到了主机发的数据了.接下来我们发过来测一下。  

   Nano发给主机
   Nano往`/dev/ttyTHS1`里写入数据。主机往`/dev/ttyUSB0`里读取数据，  
   ![Nano发给主机](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220219232003.png)  

哈哈哈，主机也收到了数据。这就表示我们之前的想法是对了。那么实现C++的串口收发还会有什么问题吗。就是文件读写。这里我就不pro出代码了，后续会放到仓库里。





