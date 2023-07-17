---
title: WebAssembly编程
date: 2023-06-02 13:48:23
categories:
- WebAssembly
tags:
- C++
- WebAssembly
- js
---

<h1 align="Center" href="https://cntofu.com/book/150/readme.html">C/C++面向WebAssembly编程</h1>

# 第一章 Emscripten入门
1. 安装Emscripten  
准备python环境
`conda create -n py2 python=2.7`
激活环境`conda activate py2`
下载emsdk
`git clone https://github.com/juj/emsdk.git`  
安装并激活`Emscripten`
Linux
```sh
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
```
Windows  
```sh
emsdk.bat install latest

# (py2) E:\emsdk>emsdk.bat install  latest
# Resolving SDK alias 'latest' to '3.1.40'
# Resolving SDK version '3.1.40' to 'sdk-releases-c3122846bb040798aab975f61008c37eb19476de-64bit'
# Installing SDK 'sdk-releases-c3122846bb040798aab975f61008c37eb19476de-64bit'..
# Installing tool 'node-15.14.0-64bit'..
# Downloading: E:/emsdk/zips/node-v15.14.0-win-x64.zip from https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/node-v15.14.0-win-x64.zip, 28255378 Bytes
# Unpacking 'E:/emsdk/zips/node-v15.14.0-win-x64.zip' to 'E:/emsdk/node/15.14.0_64bit'
# Done installing tool 'node-15.14.0-64bit'.
# Installing tool 'python-3.9.2-nuget-64bit'..
# Downloading: E:/emsdk/zips/python-3.9.2-4-amd64+pywin32.zip from https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/python-3.9.2-4-amd64+pywin32.zip, 14413267 Bytes
# Unpacking 'E:/emsdk/zips/python-3.9.2-4-amd64+pywin32.zip' to 'E:/emsdk/python/3.9.2-nuget_64bit'
# Done installing tool 'python-3.9.2-nuget-64bit'.
# Installing tool 'java-8.152-64bit'..
# Downloading: E:/emsdk/zips/portable_jre_8_update_152_64bit.zip from https://storage.googleapis.com/webassembly/emscripten-releases-builds/deps/portable_jre_8_update_152_64bit.zip, 69241499 Bytes
# Unpacking 'E:/emsdk/zips/portable_jre_8_update_152_64bit.zip' to 'E:/emsdk/java/8.152_64bit'
# Done installing tool 'java-8.152-64bit'.
# Installing tool 'releases-c3122846bb040798aab975f61008c37eb19476de-64bit'..
# Downloading: E:/emsdk/zips/c3122846bb040798aab975f61008c37eb19476de-wasm-binaries.zip from https://storage.googleapis.com/webassembly/emscripten-releases-builds/win/c3122846bb040798aab975f61008c37eb19476de/wasm-binaries.zip, 425017845 Bytes
# Unpacking 'E:/emsdk/zips/c3122846bb040798aab975f61008c37eb19476de-wasm-binaries.zip' to 'E:/emsdk/upstream'
# Done installing tool 'releases-c3122846bb040798aab975f61008c37eb19476de-64bit'.
# Done installing SDK 'sdk-releases-c3122846bb040798aab975f61008c37eb19476de-64bit'.

emsdk.bat activate latest

# (py2) E:\emsdk>emsdk.bat activate latest
# Resolving SDK alias 'latest' to '3.1.40'
# Resolving SDK version '3.1.40' to 'sdk-releases-c3122846bb040798aab975f61008c37eb19476de-64bit'
# Setting the following tools as active:
#    node-15.14.0-64bit
#    python-3.9.2-nuget-64bit
#    java-8.152-64bit
#    releases-c3122846bb040798aab975f61008c37eb19476de-64bit

# Adding directories to PATH:
# PATH += E:\emsdk
# PATH += E:\emsdk\upstream\emscripten

# Setting environment variables:
# PATH = E:\emsdk;E:\emsdk\upstream\emscripten;D:\Program Files\miniconda\envs\py2;D:\Program Files\miniconda\envs\py2\Library\mingw-w64\bin;D:\Program Files\miniconda\envs\py2\Library\usr\bin;D:\Program Files\miniconda\envs\py2\Library\bin;D:\Program Files\miniconda\envs\py2\Scripts;D:\Program Files\miniconda\envs\py2\bin;D:\Program Files\miniconda\condabin;D:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2\bin;D:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v10.2\libnvvp;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0;C:\Windows\System32\OpenSSH;C:\Program Files (x86)\NVIDIA Corporation\PhysX\Common;D:\Program Files\Bandizip;D:\Program Files\Git\cmd;D:\Program Files\TortoiseSVN\bin;C:\Program Files\dotnet;C:\Program Files\NVIDIA Corporation\NVIDIA NvDLISR;C:\Program Files\NVIDIA Corporation\Nsight Compute 2022.3.0;D:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\bin;D:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.8\libnvvp;D:\Program Files\CMake\bin;E:\nvm;E:\nodejs;D:\Program Files\miniconda;D:\Program Files\miniconda\Library\mingw-w64\bin;D:\Program Files\miniconda\Library\usr\bin;D:\Program Files\miniconda\Library\bin;D:\Program Files\miniconda\Scripts;C:\Users\nblh\AppData\Local\Microsoft\WindowsApps;D:\Program Files\Microsoft VS Code\bin;C:\Users\nblh\.dotnet\tools;C:\Program Files (x86)\Nmap;E:\flutter\bin
# EMSDK = E:/emsdk
# EMSDK_NODE = E:\emsdk\node\15.14.0_64bit\bin\node.exe
# EMSDK_PYTHON = E:\emsdk\python\3.9.2-nuget_64bit\python.exe
# JAVA_HOME = E:\emsdk\java\8.152_64bit
# Clearing existing environment variable: EMSDK_PY
# The changes made to environment variables only apply to the currently running shell instance. Use the 'emsdk_env.bat' to re-enter this environment later, or if you'd like to register this environment permanently, rerun this command with the option --permanent.

emsdk_env.bat


```  

2. 检验安装
```sh
emcc -v

# (py2) E:\emsdk>emcc -v
# emcc (Emscripten gcc/clang-like replacement + linker emulating GNU ld) 3.1.40 (5c27e79dd0a9c4e27ef2326841698cdd4f6b5784)
# clang version 17.0.0 (https://github.com/llvm/llvm-project 2922e7cd9334797c24a317d41275f1258ef9ddd3)
# Target: wasm32-unknown-emscripten
# Thread model: posix
# InstalledDir: E:\emsdk\upstream\bin

```

3. 生产简单示例`hello word`  
新建一个`main.cpp`
```C++
//main.cpp
#include <stdio.h>
int main() {
    printf("hello word！\n");
    return 0;
}
```
编译
```sh
emcc main.cpp -s WASM=1 -o main.html
```
下面列出了我们命令中选项的细节：
- -s WASM=1 — 指定我们想要的 wasm 输出形式。如果我们不指定这个选项，Emscripten 默认将只会生成asm.js。
- -o main.html — 指定这个选项将会生成 HTML 页面来运行我们的代码，并且会生成 wasm 模块，以及编译和实例化 wasm 模块所需要的“胶水”js 代码，这样我们就可以直接在 web 环境中使用了。  

这个时候在您的源码文件夹应该有下列文件：

- main.wasm 二进制的 wasm 模块代码
- main.js 一个包含了用来在原生 C 函数和 JavaScript/wasm 之间转换的胶水代码的 JavaScript 文件
- main.html 一个用来加载，编译，实例化你的 wasm 代码并且将它输出在浏览器显示上的一个 HTML 文件



# 构建从C/C++导出函数到wasm库的方法
## 方法一
1. 首先编写C/C++语言的功能处理代码
    ```c++
    ErrorCode initDecoder(int fileSize, int logLv) {
        ErrorCode ret = kErrorCode_Success;
        do {
            //Log level.
            logLevel = logLv;

            if (decoder != NULL) {
                break;
            }

            decoder = (WebDecoder *)av_mallocz(sizeof(WebDecoder));
            if (fileSize >= 0) {
                decoder->fileSize = fileSize;
                sprintf(decoder->fileName, "tmp-%lu.mp4", getTickCount());
                decoder->fp = fopen(decoder->fileName, "wb+");
                if (decoder->fp == NULL) {
                    simpleLog("Open file %s failed, err: %d.", decoder->fileName, errno);
                    ret = kErrorCode_Open_File_Error;
                    av_free(decoder);
                    decoder = NULL;
                }
            } else {
                decoder->isStream = 1;
                decoder->fifoSize = kDefaultFifoSize;
                decoder->fifo = av_fifo_alloc(decoder->fifoSize);
            }
        } while (0);
        simpleLog("Decoder initialized %d.", ret);
        return ret;
    }
    ```
2. 编写构建*.Wasm和*.js的导出脚本文件
    ```sh
    export TOTAL_MEMORY=67108864 #定义导出的Wasm库文件在运行时消耗的内存大小
    export EXPORTED_FUNCTIONS ="[ \     #定义导出函数列表，将C/C++文件里的函数方法导出到JS
        '_initDecoder', '_malloc' ]"
    emcc *.c/C++ -I ${INCLUDE_FILES} \
        -L ${LIBS_LIBRARIES} \
        -s WASM=1 #表示输出wasm的文件，默认输出asm.js
        # -s  SIDE_MODULE=1 #只输出一个文件。 只有Wasm文件生成
        -03 \
        -s TOTAL_MEMORY=${TOTAL_MEMORY} \ # 运行时消耗内存的大小
        -s EXPORTED_FUNCTIONS="${EXPORTED_FUNCTIONS}" \ # 定义导出函数的列表
        -s EXTRA_EXPORTED_RUNTIME_METHODS="['addFunction']" \ #定义导出函数在运行时的加载方式，‘addfunction’ 
        -s RESERVED_FUNCTION_POINTERS=14 \ #用于指定存储导出函数表中函数指针的大小
        -o libDecodec.js #用于指定导出函数库的名称
    ```

3. 在JavaScript中调用Wasm
    ```javaScript
    self.importScripts("libDecodec.js");
    Decoder.prototype.initDecoder=function (fileSize, chunkSize){
        var ret = Module._initDecoder(fileSize, this.coreLogLevel);
        this.logger.logInfo("initDecoder return " + ret + ".");
        if (0 == ret) {
            this.cacheBuffer = Module._malloc(chunkSize);
        }
        var objData = {
            t: kInitDecoderRsp,
            e: ret
        };
        self.postMessage(objData);
    }

    ```



## 方法二
1. 定义导出函数符号
```C++ DLL.cpp
#ifndef EM_EXPORT
#	if defined(__EMSCRIPTEN__)
#		include <emscripten.h>
#		if defined(__cplusplus)
#			define EM_EXPORT(rettype) extern "C" rettype EMSCRIPTEN_KEEPALIVE
#		else
#			define EM_EXPORT(rettype) rettype EMSCRIPTEN_KEEPALIVE
#		endif
#	else
#		if defined(__cplusplus)
#			define EM_EXPORT(rettype) extern "C" rettype
#		else
#			define EM_EXPORT(rettype) rettype
#		endif
#	endif
#endif

#include <stdio.h>

EM_PORT_API(int) GetValue() {
	return 123;
}

EM_PORT_API(float) add(float a, float b) {
	return a + b;
}
```
使用emcc编译导出
```sh
emcc DLL.cpp -o DLL.js
```  

在CMAKE中使用编译命令

```sh

set(EXPORTED_FUNCTIONS "_doWork,_main,_myFunction")
add_custom_target(
        ${PROJECT_NAME}.js
        COMMAND ${EMSDK_HOME}/emsdk activate latest
        COMMAND  ${EMSDK_HOME}/emsdk_env.bat
        COMMAND  ${EMSDK_HOME}/upstream/emscripten/emcc.bat   ${CMAKE_SOURCE_DIR}/wasmlibdemo/main.cpp
      #  -std=c++11
       # -s USE_PTHREADS=1
        -s WASM=1
        -s EXPORTED_FUNCTIONS="${EXPORTED_FUNCTIONS}"
        -s EXPORTED_RUNTIME_METHODS="['ccall']"
     #  -s RESERVED_FUNCTION_POINTERS=14
     #   -s TOTAL_MEMORY=131072000
      #  -s USE_SDL=2
        -o demoWasm.js
)

```
## 说明:
导出函数使用`EXPORTED_FUNCTIONS`与`EMSCRIPTEN_KEEPALIVE`的差异：  
1. EMSCRIPTEN_KEEPALIVE 导出符号：这是一种标记函数的宏，用于告诉编译器将该函数保留为导出函数。通过在函数声明前添加 EMSCRIPTEN_KEEPALIVE 宏，可以确保该函数被包含在编译后的 WebAssembly 模块中，并能够从 JavaScript 中访问到。
```sh

#include <emscripten.h>

EMSCRIPTEN_KEEPALIVE
int myFunction(int param) {
    // 函数的实现
    return param + 1;
}

```  
使用 EMSCRIPTEN_KEEPALIVE 标记的函数将被自动导出，不需要额外的配置或命令行选项。这种方法适用于简单的导出函数场景。  

2. EXPORTED_FUNCTIONS 定义导出函数：这是一种通过配置 Emscripten 构建系统来明确指定要导出的函数的方法。通过在构建命令中添加 EXPORTED_FUNCTIONS 选项或编写一个 .txt 文件来指定要导出的函数列表。  
```sh
emcc mycode.cpp -o mymodule.js -s EXPORTED_FUNCTIONS="['_myFunction']"
```  
这种方法允许您精确控制导出的函数，并且可以一次性导出多个函数。您可以在构建时使用 EXPORTED_FUNCTIONS 选项定义导出函数，也可以将其写入一个文本文件，然后使用 -s EXPORTED_FUNCTIONS_FILE 选项指定该文件。



创建页面测试  
```html
<!doctype html>

<html>
  <head>
    <meta charset="utf-8">
    <title>Emscripten:DLL</title>
  </head>
  <body>
    <script>
    Module = {};
    Module.onRuntimeInitialized = function() {
      console.log(Module._GetValue());
      console.log(Module._add(12, 1.0));
    }
    </script>
    <script src="DLL.js"></script>
  </body>
</html>

```



## 注意:
在 WebAssembly（Wasm）中处理 C++ 中的线程涉及一些特定的技术和策略。由于 Wasm 的单线程限制，Wasm 本身并不直接支持多线程。但是，可以使用一些技术来模拟线程行为或利用 Web Workers 来实现并发性。
下面是两种常见的处理方法：
1. 方法一
基于单线程的模拟：
- 使用异步编程模型：基于单线程的异步编程模型可以替代多线程。通过将耗时的操作分解成小任务，并使用事件轮询或回调函数进行调度，可以实现并发性和异步性。
- 使用协程：协程是一种轻量级的线程替代方案，它允许在同一个线程中创建多个独立的执行上下文。通过协程库，例如 Boost.Coroutine 或 Coroutine TS，可以实现协程以模拟线程行为。
- 使用定时器和分片：如果任务可以细分为小的计算单元，可以使用定时器和分片技术来模拟线程行为。在每个时间片中执行一小部分任务，然后切换到下一个任务。

2. 方法二
使用 Web Workers:
- Web Workers 是浏览器提供的一种机制，可以在后台运行脚本，并发地处理任务。您可以将 C++ 代码转换为 JavaScript，并将其作为 Web Worker 使用。通过将工作分配给不同的 Web Workers，您可以实现基于多个线程的并发性。
- 可以使用 Emscripten 的 Pthreads 支持将 C++ 线程代码转换为 WebAssembly，并将其封装在 Web Worker 中。