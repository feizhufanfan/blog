---
title: 内存泄露检测--valgrind
date: 2023-12-22 17:45:32
categories:
- Linux
- valgrind
tags:
- 内存泄露检测
---

<h1 align="Center">内存泄露检测————Valgrind</h1>  


## 参数

- tool 用于指定分析工具

1. Memcheck
最常用的工具，用来检测程序中出现的内存问题，所有对内存的读写都会被检测到，一切对malloc()/free()/new/delete的调用都会被捕获。所以，它能检测以下问题：  

```  

对未初始化内存的使用；
读/写释放后的内存块；
读/写超出malloc分配的内存块；
读/写不适当的栈中内存块；
内存泄漏，指向一块内存的指针永远丢失；
不正确的malloc/free或new/delete匹配；
memcpy()相关函数中的dst和src指针重叠。  

```

2. Callgrind
和gprof类似的分析工具，但它对程序的运行观察更是入微，能给我们提供更多的信息。和gprof不同，它不需要在编译源代码时附加特殊选项，但加上调试选项是推荐的。Callgrind收集程序运行时的一些数据，建立函数调用关系图，还可以有选择地进行cache模拟。在运行结束时，它会把分析数据写入一个文件。callgrind_annotate可以把这个文件的内容转化成可读的形式。  

3. Cachegrind
Cache分析器，它模拟CPU中的一级缓存I1，Dl和二级缓存，能够精确地指出程序中cache的丢失和命中。如果需要，它还能够为我们提供cache丢失次数，内存引用次数，以及每行代码，每个函数，每个模块，整个程序产生的指令数。这对优化程序有很大的帮助。  

4. Helgrind
它主要用来检查多线程程序中出现的竞争问题。Helgrind寻找内存中被多个线程访问，而又没有一贯加锁的区域，这些区域往往是线程之间失去同步的地方，而且会导致难以发掘的错误。Helgrind实现了名为“Eraser”的竞争检测算法，并做了进一步改进，减少了报告错误的次数。不过，Helgrind仍然处于实验阶段。  

5. Massif
堆栈分析器，它能测量程序在堆栈中使用了多少内存，告诉我们堆块，堆管理块和栈的大小。Massif能帮助我们减少内存的使用，在带有虚拟内存的现代系统中，它还能够加速我们程序的运行，减少程序停留在交换区中的几率。
此外，lackey和nulgrind也会提供。Lackey是小型工具，很少用到；Nulgrind只是为开发者展示如何创建一个工具。  

- leak-check  

设置这个选项时，当被测程序结束时查找内存泄露。设置为summary，Valgrind会统计有多少内存泄露发生，若设置为full、yes，Valgrind会给出每一个独立的泄露的详细信息

- error-limit  

设置为no时如果错误太多要停止显示新的错误的选项

- num-callers  
这个值默认是12，最高是50。表示显示多少层的堆栈，设置越高会使Valgrind运行越慢而且使用更多的内存,但是在嵌套调用层次比较高的程序中非常实用

- log-file  
指定检测结果报告的存放路径和文件名

## 使用示例

``` bash

Valgrind --tool=memcheck --leak-check=full --show-leak-kinds=all --show-reachable=yes --track-origins=yes
--trace-children=yes --log-file=./outLog.txt
# --tool=memcheck   # 指定分析工具
# --leak-check=full # 
# --show-leak-kinds=all 
# --show-reachable=no 
# --track-origins=yes
# –-trace-children=no #跟踪子线程
# -–log-file 输出log文件
```
