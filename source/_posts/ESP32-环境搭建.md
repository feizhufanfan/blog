---
title: ESP32--环境搭建
date: 2025-07-02 23:26:07
categories:
- esp32
- 环境搭建
tags:
- 番茄计划
- esp32
---

<h1 align="center">ESP32-clion开发环境搭建</h1>

## 环境准备
- Windows环境
  1. [IDF安装包](https://dl.espressif.cn/dl/esp-idf/?idf=4.4)
  2. [Clion](https://www.jetbrains.com/clion/)

## 环境部署（包含Clion配置）
- Windows环境
  1. 安装IDF框架
  ![安装IDF长路径支持](http://blog.feizhufanfan.top:18088/minio/images//blog/2025/07/05/20250705143501.png)  
  弹出长路径支持点击修复。
  ![路径](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705144009.png)  
  这里选择安装到F盘，后续采用默认配置等待安装完成。
  ![驱动](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705144558.png)  
  同意设备驱动安装.
  ![完成安装](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705144717.png)  
  安装完成。
  2. [CLion安装](https://www.jetbrains.com/zh-cn/clion/download/?section=windows)
  Clon的安装教程参考网上，这里不再叙述。

## 环境配置
- Windows环境  
  
  1. 打开idf框架安装所在位置。
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705145927.png)  

  2. 创建bat文件`idfDevlopEnv5.3.bat`  
 
      内容如下:
      ``` bat

      set IDF_TOOLS_PATH=C:\Espressif
      @call C:\Espressif\python_env\idf5.3_py3.11_env\Scripts\activate.bat
      @call C:\Espressif\frameworks\esp-idf-v5.3.3\export.bat
      @call C:\Espressif\idf_cmd_init.bat  

      ```  

  3. 在`C:\Espressif\python_env\idf5.3_py3.11_env\Scripts`在该路径下创建`activate.bat`文件。  
  
      内容如下:  

      ``` bat  

      @echo off

      set "VIRTUAL_ENV=C:\Espressif\python_env\idf5.3_py3.11_env"

      if defined _OLD_VIRTUAL_PROMPT (
          set "PROMPT=%_OLD_VIRTUAL_PROMPT%"
      ) else (
          if not defined PROMPT (
              set "PROMPT=$P$G"
          )
          if not defined VIRTUAL_ENV_DISABLE_PROMPT (
              set "_OLD_VIRTUAL_PROMPT=%PROMPT%"
          )
      )
      if not defined VIRTUAL_ENV_DISABLE_PROMPT (
          set "ENV_PROMPT="
          if NOT DEFINED ENV_PROMPT (
              for %%d in ("%VIRTUAL_ENV%") do set "ENV_PROMPT=(%%~nxd) "
          )
          )
          set "PROMPT=%ENV_PROMPT%%PROMPT%"
      )

      REM Don't use () to avoid problems with them in %PATH%
      if defined _OLD_VIRTUAL_PYTHONHOME goto ENDIFVHOME
          set "_OLD_VIRTUAL_PYTHONHOME=%PYTHONHOME%"
      :ENDIFVHOME

      set PYTHONHOME=

      REM if defined _OLD_VIRTUAL_PATH (
      if not defined _OLD_VIRTUAL_PATH goto ENDIFVPATH1
          set "PATH=%_OLD_VIRTUAL_PATH%"
      :ENDIFVPATH1
      REM ) else (
      if defined _OLD_VIRTUAL_PATH goto ENDIFVPATH2
          set "_OLD_VIRTUAL_PATH=%PATH%"
      :ENDIFVPATH2

      set "PATH=%VIRTUAL_ENV%\Scripts;%PATH%"
      
      ```  



## 配置环境测试

- Windows
  -  打开`CMD`并执行`C:\Espressif\idfDevlopEnv5.3.bat`  
  
     ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705151843.png)
     出现以下内容则表明配置完成。  
     ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705152207.png)  

  - 打开Clion并进入`Settings`/`Build`/`ToolChains`   
   
    点击`+`新增一个编译工具链选择`System`    
    ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705152925.png)  

    根据内容填写编译工具链  
    ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705153157.png)

    配置完成后如下
    ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705153341.png)  


## 创建工程测试  
- 激活 __IDF__ 编译环境    
  在终端当中选择 __IDF__ 标签
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705155202.png)  

  出现以下内容表示环境激活成功  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705155428.png)  

  切换到需要创建工程的目录下，这里我以`F:\WorkSpace\ESP32`为例。
  ``` shell
  cd  F:\WorkSpace\ESP32
  ```  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705155636.png)

- 创建工程  
  
  执行``命令创建工程  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705155907.png)  

  在Clion中打开该工程目录并选择已经创建的编译工具链`Esp-Idf5.3`  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705160148.png)  

  通过Clion的Cmake终端可以看出项目已经构建完成。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705160513.png)  

- 编译固件  

  在`Build`菜单栏当中选择重新重新编译  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705160601.png)  

  等待编译结束  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705160753.png)  

  编译完成  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705160822.png) 
  
  在`f:\WorkSpace\ESP32\HelloWolrd\cmake-build-debug-esp-idf53\`目录下可以看到`HelloWolrd.elf`编译好的固件。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705161206.png)

- 烧录固件  
  将esp32的USB口连接到电脑。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/1376f06814b2ab4f49cadd415fc754d.jpg)


  在设备管理器里面查看串口号。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705163222.png)

  在激活的IDF终端里面切换到工程目录。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705163501.png)  

  执行烧录命令` idf.py -p com10 app-flash`。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705163905.png)  

  出现以下内容表示烧录完成。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705163947.png)  

  执行监控命令`idf.py -p com10 monitor`查看运行状态。  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705164226.png)

  运行结果：  
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705164421.png)
  由图可知运行正常。
  ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705164358.png)

### 补充
-  __停止监听__
   在终端里面输入<kbd>Ctrl</kbd>+<kbd>]</kbd>来实现停止监听
   ![](http://blog.feizhufanfan.top:18088/minio/images/blog/2025/07/05/20250705164826.png) 
