---
title: STM32--使用Clion开发
date: 2023-07-17 13:52:35
categories:
- STM32
- Clion开发
tags:
- STM32
- Clion
---

<h1 align="center">使用CLion优雅の开发STM32</h1>


# 环境准备
1. [Clion](https://www.jetbrains.com/clion/download/other.html) 2019版本以上吧
2. [STM32cubeMX](https://www.st.com/zh/development-tools/stm32cubemx.html#get-software) 推荐6.5版本以上，经过测试5.0~6.4存在更新STM32Pack无法连接网络的问题。
3. [mingw64](https://www.aliyundrive.com/s/WH4uzN1gHgu) mingw64中的gcc版本最好和arm-gun-abi-gcc的版本保持一致，这里我使用[mingw64-gcc9.1](https://sysprogs.com/getfile/513/mingw32-gcc9.1.0.exe)
4. [gcc-arm-none-eabi-9-2019-q4-major-win32](https://developer.arm.com/-/media/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-win32.zip?revision=20c5df9c-9870-47e2-b994-2a652fb99075&rev=20c5df9c987047e2b9942a652fb99075&hash=DBEB34DE4AB3A1EB549D64C02F2E426080226698)
5. [openOCD](https://sysprogs.com/getfile/2052/openocd-20230621.7z) 



# 环境配置
- `mingw64`安装配置
1. 安装mingw64（如果通过其他方式已经安装好，可以跳过该小节）  

![mingw64-gun-tool](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717144805.png)  
这里先将添加到环境变量的选项去除稍后通过手动配置。
等待安装完成。
2. 配置`mingW64`到环境变量
在系统环境变量里新建一个`GCC_HOME`，将MingW64的安装目录填入
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717145719.png)
然后在找到path
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717145933.png)
然后在末尾添加`%GCC_HOME%/bin`
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717150317.png)
3. 验证GCC是否安装成功
按住'Ctrl'+'R'输入cmd，回车新建一个cmd窗口。输入命令`gcc -v`
出现一下内容则表示安装成功
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717150445.png)

- `gcc-arm-none-eabi`安装配置
1. 下载好的安装包解解压到想要安装的目录，这里以`E:\gcc-arm-none-eabi-9-2019-q4-major-win32`为例。**提示**安装目录路径中最好**不要**出现**空格**或者**中文**。
2. 配置`gcc-arm-none-eabi`到环境变量
在系统环境变量里新建一个`ARMGCC_HOME`，将安装路径填入
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717151600.png)
然后在找到path，在末尾添加一个`%ARMGCC_HOME%\bin`
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717151823.png)
3. 验证GCC是否安装成功
按住'Ctrl'+'R'输入cmd，回车新建一个cmd窗口。输入命令`arm-none-eabi-gcc -v`
出现一下内容则表示安装成功
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717151953.png)

至于其他安装的安装过程就省略啦。 软件安装搞开发的基本都没啥大问题是吧~ 帅比靓女们、

# 配置CLion
1. 打开CLion，回到最开始的首页点击自定义选择里面的全部设置。
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717152642.png)
2. 选择插件/Plugins，确认里面的`Embedded MCU Development plugin`的插件是开启。 
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717153209.png)
3. 在`Build`中选择`Toolchain`，新建一个MinGW_ARM的工具链配置，将MINGw、GCC、G++、GDB配置到相应的安装目录
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717153711.png)
4. 在`BUild`中选择`Embedded Development`,在`openOCD`以及`STM32cubeMX`中填入对应的安装路径
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717154234.png)
5. 测试是否配置成功
分别点击`openOCD`以及`STM32cubeMX`对于的Test，出现如图信息则表示配置成功。
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717154507.png)  
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717154531.png)

# STM32cubeMX配置
1. 打开STM32cube，选择安装软件包
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717154916.png)
2. 根据自己的需要安装对应的软件包，这里我使用的事STM32F103系列的，选择STM32F1
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717155216.png)
3. 安装完成关闭整个cubeMX软件。


# 建立测试工程
1. 在CLion的选择新建项目，选择STM32cubeMX项目类型，选择项目建立位置。  **提示**记录该软件位置。
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717155547.png)
2. 等待工程建立成功。
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717155715.png)
3. 跳转到`STM32cubeMX`界面
工程建立好了之后，点击`*.ioc`文件在编辑窗口里点击打开完成`STM32cubeMX`完成窗口的跳转。
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717160052.png)
4. 配置`STM32开发板`的相关参数
- 点击选择开发版
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717161145.png)
- 选择相应的芯片类型，这里以`STM32F103C6T6`为例
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717161358.png)
- 生成工程代码
在引脚、时钟等配置好了之后点击`Project`,在工程名称填入之前Clion里创建工程的名称**切记保持名称一致**，然后在`Toolchain/IDE`里选择`STM32cubeIDE`。
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717161831.png)  

接下来点击`CODE generator`在里面配置代码生成的项目细节，这里选择头文件分离。  

![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717162229.png)  



最后点击`Generator Code`生成代码等待生成结束后。  

![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717162514.png)  

在弹出的提示框里，选择yes  

![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717162631.png)  

点击关闭按钮并关闭STM32cubeMX软件。

![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717162723.png)  


5. 回到Clion里等待项目更新。
在项目更新结束后，出现的弹框里选择跳过。因为后续我们需要对其进行重新配置。
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717162950.png)

最后这便是建立好的工程
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717163045.png)
# 编译
为工程选择我们创建的`MinGW_ARM`工具链
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717172646.png)

点击编译按钮
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717163206.png)
等待结束
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717163322.png)
从编译提示框里可以看出已经从成功编译出`test.bin`、`test.elf`、`test.hex`。


# 下载&烧录
- 配置下载与烧录
点击如图所示的位置
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717163756.png)
在弹出的窗口里选择添加一个`openOCD Download & Run`
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717163959.png)
在相应位置填入对应参数
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717164320.png)
在点击开发板配置文件时弹出的配置文件选择框里面选择一个接近的类型，我的板子为`STM32F103C6T6`因此选择了如图所示的类型。
其实这个文件主要对应的是下载方式以及对应的Flash大小。这里选择这个配置文件是不用自己再去创建，通过这个文件去修改
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717165003.png)
选择应用之后，回到工程界面点击刚刚选择的配置文件
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717165425.png)

- 开发板下载烧录配置文件
这里根据自己使用的下载器的实际情况选择以下两种
1. 拼多多或者淘宝买  
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717170154.png)
```
# choose st-link/j-link/dap-link etc.
adapter driver cmsis-dap
transport select swd
​
# 0x10000 = 64K Flash Size
set FLASH_SIZE 0x20000
​
source [find target/stm32f1x.cfg]
​
# download speed = 10MHz
adapter speed 10000 
```
2. 官方的ST-link
```
# choose st-link/j-link/dap-link etc.
#adapter driver cmsis-dap
#transport select swd
source [find interface/stlink.cfg]
transport select hla_swd
source [find target/stm32f1x.cfg]
# download speed = 10MHz
adapter speed 10000

```

这里我使用的第一种下载器同时板子(STM32F103C6T6)的`ram`和`flash`与STM32F103C8T6是不一样的从前面编译结果可以得知`Ram`与`Flash`分别为`10kb`、`32kb`。 因此我修改配置文件如下
```
# choose st-link/j-link/dap-link etc.
adapter driver cmsis-dap
transport select swd
​
# 0x10000 = 64K Flash Size
set FLASH_SIZE 0x8000
​
source [find target/stm32f1x.cfg]
​
# download speed = 10MHz
adapter speed 10000 
```
![](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230717171135.png)


补充：
如果对自己的芯片不知道怎么设置，可以参考OpenOCD自带的一系列配置文件，路径在OpenOCD安装目录的`share\openocd\scripts`下：
![借用稚晖大佬的图](https://pic3.zhimg.com/80/v2-accd234d51121834b43397c8a20c12a6_720w.webp)

只需要关注这几个目录：
- **board**：板卡配置，各种官方板卡
- **interface**：仿真器类型配置，比如ST-Link、CMSIS-DAP等都在里面
- **target**：芯片类型配置，STM32F1xx、STM32L0XX等等都在里面
在配置文件中不要加reset_config srst_only这一句，会导致下载失败，这一句是指示系统重启的，删除不影响下载。

接下来便可以下载调试代码了。

后续更新在没有板子和下载器的情况下进行简单的开发调试。

文章参考：[配置CLion用于STM32开发【优雅の嵌入式开发】](https://zhuanlan.zhihu.com/p/145801160)








