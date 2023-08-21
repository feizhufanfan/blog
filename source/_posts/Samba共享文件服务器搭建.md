---
title: Samba共享文件服务器搭建
date: 2023-08-18 22:33:06
categories:
- linux
- samba
tags:
- linux
- samba
---

# 安装SAMBA服务
```
sudo apt install samba

```

## 配置SAMBA服务
1. 创建访问账户  
``` bash
sudo useradd smbuser -s /usr/sbin/nologin
# /usr/sbin/nologin 创建不需要登陆的账户用于samba文件分享
```

2. 修改共享的文件夹权限
``` bash

sudo chmod +axr /home/work/sharedir

```

3. 将用户smbuser添加到samba的smbpasswd file中（即在samba服务中注册该账户）
``` bash

sudo smbpasswd -a smbuser
#后续设置登录密码，用于远程访问

```

4. 修改samba配置文件（/etc/samba/smb.conf）
``` bash

# 打开文件
sudo vim /etc/samba/smb.conf
#在文件尾部添加以下信息，并保存（vim中:wq保存）
 

#共享目录名，访问时的展示名
[secret]  
#该共享目录的描述
    comment = Secret File    
#访问的实际路径,前面设置的  
    path = /home/work/sharedir 
#设置可访问的用户，此处为前面添加的用户smbuser（注意users不要拼写错误）
    valid users = smbuser     
#是否允许访客，否 
    guest ok = no     
#可写，是        
    writable = yes  
#可浏览，是          
    browsable = yes           

```

5. 重启服务，使上述设置生效
``` bash

sudo service smbd restart
sudo service nmbd restart
#或者以下方法
sudo restart smbd
sudo restart nmbd


```

# 辅助命令
``` bash
# 查看samba用户列表
pdbedit -L
 
# 对samba用户进行管理（用户已经在系统中创建）
smbpasswd -h  #查看支持的命令列表
 
# 异常时可查看日志情况
cat /var/log/samba/log.%m

```


## 补充：如需开启匿名访问
``` bash

[share] 
    comment = Ubuntu File Server 
    path = /home/work/shareAll 
    browsable = yes 
    guest ok = yes 
    read only = no


```
