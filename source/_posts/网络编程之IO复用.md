---
title: 网络编程之IO复用
date: 2023-05-12 15:33:21
categories:
- 网络编程
- IO复用
tags:
- 网络IO复用
---

# 阻塞 IO
服务端为了处理客户端的连接和请求的数据，写了如下代码  
``` c++
listenfd = socket();   // 打开一个网络通信端口
bind(listenfd);        // 绑定
listen(listenfd);      // 监听
while(1) {
  connfd = accept(listenfd);  // 阻塞建立连接
  int n = read(connfd, buf);  // 阻塞读数据
  doSomeThing(buf);  // 利用读到的数据做些什么
  close(connfd);     // 关闭连接，循环等待下一个连接
} 
```

这段代码会执行得磕磕绊绊，就像这样。
![阻塞 IO](https://p6-tt-ipv6.byteimg.com/img/pgc-image/cf4ef70d55034709ab7f4070daa71e40~tplv-obj.image)

可以看到，服务端的线程阻塞在了两个地方，一个是 accept 函数，一个是 read 函数。
如果再把 read 函数的细节展开，我们会发现其阻塞在了两个阶段。
![](https://p6-tt-ipv6.byteimg.com/img/pgc-image/9ad4eaae0af14e5fa338575dab03ec68~tplv-obj.image)

这就是传统的阻塞 IO。
整体流程如下图。
![](https://p1-tt-ipv6.byteimg.com/img/pgc-image/99c43f1cd745489a97d3087cb17c0ff4~tplv-obj.image)
所以，如果这个连接的客户端一直不发数据，那么服务端线程将会一直阻塞在 read 函数上不返回，也无法接受其他客户端连接。

# 非阻塞 IO
为了解决上面的问题，其关键在于改造这个 read 函数。
有一种聪明的办法是，每次都创建一个新的进程或线程，去调用 read 函数，并做业务处理。
```c++
while(1) {
  connfd = accept(listenfd);  // 阻塞建立连接
  pthread_create（doWork);  // 创建一个新的线程
}
void doWork() {
  int n = read(connfd, buf);  // 阻塞读数据
  doSomeThing(buf);  // 利用读到的数据做些什么
  close(connfd);     // 关闭连接，循环等待下一个连接
}
```
这样，当给一个客户端建立好连接后，就可以立刻等待新的客户端连接，而不用阻塞在原客户端的 read 请求上。
![](https://p9-tt-ipv6.byteimg.com/img/pgc-image/6e0b80ab2bd04601a612d9ee115a1581~tplv-obj.image)













































[文章参考](https://www.cnblogs.com/flashsun/p/14591563.html):https://www.cnblogs.com/flashsun/p/14591563.html