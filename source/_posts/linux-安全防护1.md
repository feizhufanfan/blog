---
title: linux--安全防护1
date: 2025-08-11 11:50:44
categories:
- linux
- fail2ban
tags:
- linux
- 网络安全
- fail2ban
---

# Linux网络安装————Fail2ban
## 1. 工具介绍
Fail2ban 是一款用于防止暴力破解和恶意登录尝试的开源工具，广泛应用于 Linux 系统。它通过监控日志文件来检测失败的登录尝试或其他可疑活动，并自动更新防火墙规则以阻止恶意 IP 地址。

它通过监控日志文件（如 /var/log/auth.log）来检测失败的登录尝试，当某个 IP 地址在短时间内多次尝试失败后，Fail2ban 会自动将其添加到防火墙的黑名单中，阻止其进一步访问。



## 2. 安装配置
###  依赖条件
1.  检查ufw是否安装且启动
    ```bash
    sudo ufw status
    ```


如果出现以下内容则表示`ufw`启动且运行。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250813132304.png)

### 安装fail2ban
1.  更新源
    ```bash
    sudo apt update
    ```

2.  安装fail2ban
    ```bash
    sudo apt install fail2ban -y
    ```
3.  修改配置文件
    ```bash
    sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    ```

4.  新增sshd防御规则   
    ```bash
    sudo vim /etc/fail2ban/jail.local
    # 在vim界面输入
    :/ban
    # 找到banaction项
    # 修改为
    banaction = ufw # 使用ufw作为规则触发器

    # 新增sshd的防御过滤器
    [sshd]

    # To use more aggressive sshd modes set filter parameter "mode" in jail.local:
    # normal (default), ddos, extra or aggressive (combines all).
    # See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
    #mode   = normal
    enabled = true
    maxretry  = 3
    findtime  = 1d
    bantime   = 1w
    port    = 2233
    logpath = %(sshd_log)s
    backend = %(sshd_backend)s
    ```

![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250813133014.png)

**说明**:
    -   enabled : 启用该规则
    -   maxretry : 最大失败次数
    -   findtime : 查找时间 s、m、h、d 默认为秒
    -   bantime : 禁止ip时间

![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250813133444.png)

1.  启动fail2ban
```bash
sudo systemctl start fail2ban.service # 启动服务
sudo systemctl enable fail2ban.service # 设置开机启动
sudo systemctl status fail2ban.service # 查看服务运行状态
```
以下表示运行中:
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250813134426.png)

