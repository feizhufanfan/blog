---
title: STM32-TIM的使用
date: 2023-07-24 16:43:15
categories:
- STM32
- TIM
tags:
- STM32
- 番茄计划
---

# TIM的使用
在STM32cubeMX中启用TIM组件，勾选对应的中断事件。这里我使用TIM1的通用定时器功能，启用计数更新事件。点击`code Generation`生成代码。
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230725095934.png)

## 启用定时器中断
在`tim.c`文件中`MX_TIM1_Init`初始化的地方在最后一行添加`HAL_TIM_Base_Start_IT(&htim1);`完成定时器中断函数的配置。
从`TIM1_UP_IRQHandler`中断函数中可以用调用一个`HAL_TIM_IRQHandler`中断函数的句柄.
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230725101245.png)
跟踪进去可以看到在对应的事件中调用了一个`HAL_TIM_PeriodElapsedCallback`函数。
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230725101525.png)
再跟踪进去可以看到是一个弱声明。因此我们需要自己去实现，
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230725101650.png)
回到main.c文件中在main函数的上方声明实现一下。
```c++
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim){
    if (htim->Instance==TIM1){
        //do some 
    }
}

```
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20230725102026.png)






