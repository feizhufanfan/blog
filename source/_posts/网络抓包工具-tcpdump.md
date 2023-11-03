---
title: 网络抓包工具--tcpdump
date: 2023-11-03 09:17:44
categories:
- Linux
- Tcpdump
tags:
- 网络抓包工具
---

1. 抓取所有网络包，并在terminal中显示抓取的结果，将包以十六进制的形式显示。
`tcpdump`  
2. 抓取所有的网络包，并存到 result.cap 文件中。  
`tcpdump -w result.cap`
3. 抓取所有的经过eth0网卡的网络包，并存到result.cap 文件中。  
`tcpdump -i eth0 -w result.cap`
4. 抓取源地址是192.168.1.100的包，并将结果保存到 result.cap 文件中。  
`tcpdump src host 192.168.1.100 -w result.cap`
5. 抓取地址包含是192.168.1.100的包，并将结果保存到 result.cap 文件中。  
`tcpdump host 192.168.1.100 -w result.cap`
6. 抓取目的地址包含是192.168.1.100的包，并将结果保存到 result.cap 文件中。  
`tcpdump dest host 192.168.1.100 -w result.cap`
7. 抓取网卡eth0上所有包含端口22的数据包  
`tcpdump -i eth0 -vnn port 22`
8. 抓取指定协议格式的数据包，协议格式可以是「udp,icmp,arp,ip」中的任何一种,例如以下命令  
`tcpdump udp  -i eth0 -vnn`





