---
title: 通信协议--MQTT
date: 2023-08-17 14:19:09
categories:
- 通信协议
- MQTT
tags:
- MQTT
---

# 什么是MQTT
MQTT（Message Queue Telemetry Transport）是一种轻量级的消息传输协议，旨在为物联网设备提供可靠、高效的通信机制。它最初由IBM开发，现已成为ISO标准。
MQTT被设计用于低带宽、不稳定的网络环境中，适用于各类物联网应用和传感器网络。它采用发布-订阅（publish-subscribe）模式，其中包括发布者、订阅者和代理服务器。
在MQTT中，发布者负责将消息发布到特定的主题（topics），而订阅者则对感兴趣的主题进行订阅。代理服务器（broker）充当中介，负责接收发布者的消息并将其传递给订阅者。


# 协议特点
- 简单轻量：协议设计简洁，非常适合于资源受限的设备。
- 低带宽消耗：MQTT的消息头较小，有效减少网络流量和传输延迟。
- 异步通信：发布者和订阅者之间的通信是异步的，允许设备按需发送和接收消息。
- 可靠性：MQTT支持QoS（Quality of Service）级别，确保消息的可靠传输和适应性。
- 灵活性：支持动态的主题订阅和多个订阅者的扩展。

# 使用场景
1. 物联网（IoT）应用：MQTT被广泛用于物联网设备之间的通信。例如，传感器、监控设备、智能家居设备等可以使用MQTT发布传感数据、接收指令或与其他设备进行交互。
2. 远程监控和遥测：MQTT可用于监控和控制分布式设备，如能源管理系统、智能农业、智能城市等。通过MQTT，可以实时获取设备状态、传输数据并执行远程操作。
3. 移动应用推送：移动应用程序可以使用MQTT作为推送通知的机制。服务端可以使用MQTT将消息推送给订阅者，从而实现实时的消息推送功能。
4. 即时通讯：MQTT也可以用作即时通讯协议，支持实时聊天和消息传递。在这种场景下，用户可以订阅感兴趣的主题，并实时接收其他用户发送的消息。
5. 传感器数据采集：由于MQTT的轻量级和低带宽消耗，它非常适合用于传感器数据的采集和传输。通过MQTT，传感器可以将实时数据发布到特定的主题，供其他设备或应用程序订阅并进行进一步处理。
6. 实时监控和反馈：MQTT可用于实时监控系统，例如交通监控、工业设备监测等。设备可以通过MQTT发布监测数据，监控中心和相关方可以订阅这些数据以及发送反馈指令。



# 搭建MQTT服务器
## 编译安装
``` bash

git clone https://github.com/hui6075/mosquitto-cluster.git
cd mosquitto-cluster
# 这里默认不开启集群  如需开启在后面追加添加 -DWITH_CLUSTER=yes
cmake -G "Ninja" -B build -DCMAKE_BUILD_TYPE=release -DCMAKE_INSTALL_PREFIX=$(pwd)/install  .
cd build & ninja -j4
ninja install

```

## 修改配置文件
``` bash

cd install
vim etc/
## 修改为一下
# 节点名称根据自己情况修改
node_name 213_node
# ip以及端口自定义
node_address 172.19.82.213:6001
node_keepalive 60



```
![](http://feizhufanfan.top:18088/minio/images/blog/20230825145440.png)

## 部署
`./sbin/mosquitto -c ./etc/mosquitto/mosquitto.conf &`
完成部署

# 测试MQTT服务器
## 安装测试客户端
下载安装[MQTTX](https://mqttx.app/zh/downloads)  

## 安装完成之后启动软件 
新建连接并填入对于的MQTT服务器地址
![](http://feizhufanfan.top:18088/minio/images/blog/20230825150100.png)

## 测试消息
新增订阅消息类型
![](http://feizhufanfan.top:18088/minio/images/blog/20230825150248.png)

发布消息
![](http://feizhufanfan.top:18088/minio/images/blog/20230825150347.png)

测试通过



# 客户端实现MQTT代码示例
``` c++ Client.cpp

#include <iostream>
#include <cstring>
#include <mqtt/client.h>

const std::string SERVER_ADDRESS("tcp://mqtt.example.com:1883");
const std::string CLIENT_ID("mqtt_cpp_example");
const std::string TOPIC("my_topic");

int main() {
    mqtt::client client(SERVER_ADDRESS, CLIENT_ID);

    mqtt::connect_options connOpts;
    connOpts.set_keep_alive_interval(std::chrono::seconds(20));
    connOpts.set_clean_session(true);

    // 连接到MQTT代理服务器
    try {
        client.connect(connOpts);
        std::cout << "连接成功" << std::endl;
    } catch (const mqtt::exception& exc) {
        std::cerr << "连接失败: " << exc.what() << std::endl;
        return 1;
    }

    // 订阅主题
    client.subscribe(TOPIC, 0);

    // 异步接收消息并处理
    client.start_consuming();
    client.set_callback([&](const mqtt::const_message_ptr& msg) {
        // 在这里处理接收到的消息
        std::cout << "收到消息: " << msg->get_payload_str() << std::endl;
    });

    // 发布消息
    std::string payload = "Hello, MQTT!";
    mqtt::message_ptr pubmsg = mqtt::make_message(TOPIC, payload);
    client.publish(pubmsg);

    // 等待用户退出程序
    std::cout << "按下回车键退出..." << std::endl;
    std::cin.ignore();

    // 断开连接
    client.stop_consuming();
    client.unsubscribe(TOPIC);
    client.disconnect();

    return 0;
}


```
补充说明：
该示例使用[Eclipse Paho C++](https://github.com/eclipse/paho.mqtt.cpp)库。








