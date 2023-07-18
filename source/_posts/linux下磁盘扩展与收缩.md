---
title: linux下磁盘扩展与收缩
date: 2022-03-02 14:18:56
categories:
- Linux
- 磁盘管理
tags:
- lvm2
- Linux
---

<h1 align="center">磁盘扩展与收缩</h1>



## 使用工具
- lvm2
- fdisk
- df

一般linux系统在安装完成之后都有lvm2磁盘管理工具,具体的表现为pvcreate、vgs、vgdisplay等工具。没有的话使用命令直接安装Debain/Ubuntu :`sudo apt install lvm2`，CentOS/RED hat:`sudo yum install lvm2`.  

关键名词扫盲：
+ PV（Physical Volume）- 物理卷
物理卷在逻辑卷管理中处于最底层，它可以是实际物理硬盘上的分区，也可以是整个物理硬盘，也可以是raid设备。

+ VG（Volumne Group）- 卷组
卷组建立在物理卷之上，一个卷组中至少要包括一个物理卷，在卷组建立之后可动态添加物理卷到卷组中。一个逻辑卷管理系统工程中可以只有一个卷组，也可以拥有多个卷组。

+ LV（Logical Volume）- 逻辑卷
逻辑卷建立在卷组之上，卷组中的未分配空间可以用于建立新的逻辑卷，逻辑卷建立后可以动态地扩展和缩小空间。系统中的多个逻辑卷可以属于同一个卷组，也可以属于不同的多个卷组

+ PE（Physical Extent）- 物理块

## 磁盘扩展
1. 对于需要扩展的分区
    首先创建物理磁盘分区  
    `sudo fdisk /dev/sdx  #sdx新挂载的盘物理磁盘创建分区`  
    格式化文件系统  
    `sudo mkfs.ext4 /dev/sdxX`
    创建物理卷  
    `pvcreate /dev/sdxX #sdx新挂载的物理磁盘 X磁盘分区号   可以同时创建多个物理卷`  
    扩展VG 把增加的分区添加到需要增加的分区
    `vgextend xxx /dev/sdxX`
    分区空间合并  
    `lvextend -L +64G xxx /dev/sdxX #xxx待合并的文件目录（分区）`
    开启合并  
    `xfs_growfs xxx #xxx带扩展目录或分区`

## 磁盘收缩








## 多分区挂载在同一个目录
首先创建物理卷  
`pvcreate /dev/sdx1 /dev/sdx2 /dev/sdxX`  
创建VG组卷  
`vgcreate lvm_extendDisk /dev/sdx1 /dev/sdx2 /dev/sdxX`  
创建LV逻辑卷  
`lvcreate -L 100G -n lvm_LVDkisk lvm_extendDisk`  
挂载到目录  
`mount /dev/mapper/lvm_LVDisk xxx  #待挂载的目录`  






