---
title: Linux系统创建开机启动服务
date: 2022-02-18 13:37:01
categories:
- Linux
- 开机自启服务
tags:
- Linux
- 开机自启
- Ubuntu/Debian
- CentOS/UOS
- update-rc.d
- chkconfig
---  

<h1 align="center">linux下如何创建开机自启服务 </h1>

## 背景
有时候我们自己写了一些服务程序，需要在开机的时候让他们自动的运行起来。并且有时候还需要根据服务的依赖规定启动顺序。

## 实现
首先我们需要明确我们的系统是什么类型的，因为不同系统设置开机启动的服务是不一样的。
    ```sh
    uname -a
    ```
    ![uname -a](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20220218161322.png)

## Ubuntu、Debain
1. 首先编写开机启动脚本,内容模板如下:  
   `vim demo_server`
   ```sh
   #!/bin/bash
   ### BEGIN INIT INFO
   # Provides:          scriptname #服务名称 最好和开机服务脚本名称对应（demo_server）
   # Required-Start:    $remote_fs $syslog
   # Required-Stop:     $remote_fs $syslog
   # Default-Start:     2 3 4 5
   # Default-Stop:      0 1 6
   # Short-Description: Start daemon at boot time
   # Description:       Enable service provided by daemon.
   ### END INIT INFO
   demoServePath=xxxx/demoServer.sh
   case "$1" in
      start)
      echo "启动应用"
      sh $demoServePath
      ;;
      stop)
      echo "结束应用"
      killall $(basename $demoServePath)
      ;;
      *)
      echo "Usage:$0 {start|stop|reload|restart|status}"
      exit 1
   esac
   ```
2. 将编写好服务脚本复制到`/etc/init.d`目录下  
   `sudo cp demoServer /etc/init.d/`  

3. 赋予脚本运行权限  
   `sudo chmod +x /etc/init.d/demoServer`  

4. 将脚本加入系统的开机启动服务并设置启动优先级(这里有两种方式)
   ```sh
   cd /etc/init.d
   #方式一只设定启动的优先级但不设置运行的系统级别
   sudo update-rc.d demoServer defaults 90 #优先级0~90 数字越小优先级越高优先执行
   #方式二设定启动的优先级同时设定运行的基本，或设定关机时运行的级别
   sudo update-rc.d demoServer start 90 2 3 4 5 #stop 60 0 1 6
   ```

5. 启动开机服务脚本  
   `sudo update-rc.d demoServer enable`

6. 查看服务列表  
   `sudo service --status-all`
7. 服务的控制命令
   ```sh
   sudo service xxx status #查看服务状态
   sudo service xxx start  #启动服务
   sudo service xxx stop   #停止服务
   sudo service xxx restart #重启服务
   ```

8. 服务的停止与移除
   ```sh
   sudo update-rc.d  demoServer disable|enable  
   sudo update-rc.d -f demoServer remove
   ```

## CentOS7~8/UOSServer
1. 编写开机脚本demoServer  
   `vim demoServer`
   ```sh
    #!/bin/bash
    # chkconfig: 2345 90 60
    # Default-Start:     2 3 4 5
    # Default-Stop:      S 0 1 6
    # description: Saves and restores system entropy pool for \ 
    # higher quality random number generation
    servername=myservice #服务名称
    serverdir=/opt/myservice
    binpath=/opt/myservice/myservice.sh

    prog=$(basename $binpath)

    restart() {
        stop
        start
    }
    reload() {
        stop
        start
    }
    start() {
    echo -n $"Starting $daemon:"
        daemon $binpath start
        retval=$?
        echo
        [ $retval -eq 0 ]
    }

    stop() {
    echo -n $"Stopping $daemon:"
        daemon $binpath stop
        retval=$?
        echo
        [ $retval -eq 0 ]
    }

    ha_status() {
        #status $prog
        status $prog
        ps -ef|grep $prog && exit 0
    }

    case "$1" in
        start)
        $1
        ;;
        stop)
        $1
        ;;
        reload)
        $1
        ;;
        restart)
        $1
        ;;
        status)
         ha_status
        ;;
        *)
         echo "Usage:$0 {start|stop|reload|restart|status}"
         exit 1
         ;;
    esac
   ```
    - 服务脚本配置说明  
      #chkconfig:2345 90 60    #2345代表着启动的系统层级rc0~rc5，90 代表服务的启动等级越大表示服务越靠后，60 表示服务关闭时的等级越大越晚执行。
      ##chkconfig:2345 90 60这句话时CentOS、UOSServer系统使用chkconfig服务配置启动服务检查的数据段。必须要在脚本里面。
    - 脚本参照[链接](https://blog.csdn.net/zjy900507/article/details/82699694)
2. 复制脚本到/etc/init.d目录  
   `sudo cp demoServe /etc/init.d/`
3. 赋予脚本运行权限  
   `sudo chmod +x demoServer`
4. 加入系统开机启动服务  
   `sudo chkconfig --add demoServe`
5. 启动服务  
   `sudo chkconfig demoServer on`
6. 服务的停止与移除
   ```sh
   sudo chkconfig demoServer on|off
   sudo chkconfig --del demoServer
   ```

