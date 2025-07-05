---
title: STM32-RT-thread nano移植
date: 2023-07-24 16:47:18
categories:
- STM32
- RT-Thread-nano移植
tags:
- STM32 
- RT-Thread-nano
---

# STM32移植RT-Thread Nano


## 环境准备
- STM32CubeMX添加RT-thread组件包
1. 在软件界面的`Help`中点击`Manage embedded software packages`
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724170951.png)
2. 点击`From URL`
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724171140.png)
3. 添加`RT-thread Nano`组件源地址` https://www.st.com/en/development-tools/stm32cubemx.html`
在弹出的窗口中新建一个，将地址输入进去后点击`Check`检查地址的有效性。结果如下图
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724171627.png)
4. 安装组件
地址添加成功后点击`RealRhread`，选择`RT-thread`下拉选项安装对应的组件版本，安装成功后点击刷新等待刷新完成。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724172159.png)
关闭软件。


## 工程创建
- 在Clion中新建工程，跳转到STM32CubeMX后选择对应的芯片类型。
具体过程参考[STM32-使用Clion开发](https://feizhufanfan.top/2023/07/17/STM32-%E4%BD%BF%E7%94%A8Clion%E5%BC%80%E5%8F%91/)

- 在工程中添加RT-thread组件
1. 回到`STM32CubeMX`，点击`SoftWave Packts`中的`Select Componets`
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724172905.png)
2. 启用组件,根据自身情况选择对应组件点击`OK`确定启用。由于我使用的是`STM32F103C6T6` `RAM`和`Flash`都比较小，因此就不启用`device`框架了
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724173145.png)
3. 导入组件点击`midisoftWave and SoftWave Packts`,在右侧把启用的组件包勾选上。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724173616.png)
4. 将系统的硬件中断权限交给RT-thread，在`System Core`中点击`NVCC`切换到`code Genration`页面把`Hard fault interrupt`去除勾选状态。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724174020.png)
5. 生成工程
在配置完以上信息后，自此配置结束，点击`Generation`生成工程。工程具体的配置信息参考[STM32-使用Clion开发](https://feizhufanfan.top/2023/07/17/STM32-%E4%BD%BF%E7%94%A8Clion%E5%BC%80%E5%8F%91/)。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724174550.png)


## 修改STM32的启动入口
在CLion的工程界面找到`startup_stm32f103c6tx.s`文件。**提示：不同开发板的启动文件名称是不一样的，但文件所在位置一致。**
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724174829.png)
打开`startup_stm32f103c6tx.s`文件并找到以下代码段
```
/* Call the application's entry point.*/
  bl main
  bx lr
.size Reset_Handler, .-Reset_Handler

```
修改为
```
/* Call the application's entry point.*/
  bl entry
  bx lr
.size Reset_Handler, .-Reset_Handler

```

## 取消多余的时钟初始化代码
打开`main.c`文件，可以看到文件中已经有了一个示例代码
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724175512.png)
把图中标出的`HAL_Init()`、`SystemClock_Config()`函数删除或者注释




## 可能出现的问题
- 出现` error: #error Please uncomment the line <#include "finsh_config.h"> in the rtconfig.h`错误
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724175916.png)
解决办法：
在`rtconfig.h`文件中将`#include "finsh_config.h"` 注释放开
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230724180024.png)

- 出现`error: unknown type name ’UART_HandleTypeDef‘`错误
出现改错误是在创建工程项目时，没有启用串口组件导致的。在RT-Thread中默认使用`Usart1`作为`shell`控制台的输出。
解决办法：
启用Usart组件。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230725093909.png)
补充说明：
由于在启用Usart组件，RT-Thread中会被重复定义，因此需要对代码做出部分修改。
首先在`borad.c`文件中将`Usart2`改为`Usart1`。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230725094757.png)
在`void rt_hw_board_init(void)`函数实现的上面声明一个`static int uart_init(void);`,然后在`rt_hw_board_init`函数内部的` HAL_SYSTICK_Config(HAL_RCC_GetHCLKFreq()/RT_TICK_PER_SECOND); `位置新增`uart_init`的调用。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20230725095423.png)
在`main.c`文件中找到`main`函数取消其中`MX_USART1_UART_Init`函数的调用。




