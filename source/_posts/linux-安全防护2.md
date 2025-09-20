---
title: linux--安全防护2
date: 2025-09-02 18:40:36
categories:
- linux
- fail2ban
tags:
- linux
- 网络安全
- fail2ban
---

# Fail2ban邮件预警
## 环境配置
### 1.  邮件客户端安装
```bash
sudo apt update
sudo apt install -y mailutils postfix libsasl2-modules
```
### 2.  安装过程中选择：
-   **邮件配置类型：**选择 "Internet Site"
-   **系统邮件名称：**输入您的域名（如无域名可保留默认）

### 3.  配置 Postfix 使用 163/QQ 邮箱
#### 编辑主配置文件：
```bash
sudo vim /etc/postfix/main.cf
```

#### 在文件末尾添加以下内容（根据邮箱类型选择）：
1.  163 邮箱配置：
```conf
# 基本设置
myhostname = your-server-hostname  # 替换为您的服务器主机名
mydomain = your-domain.com         # 替换为您的域名
myorigin = $mydomain
relayhost = [smtp.163.com]:465
# SASL认证设置
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_mechanism_filter = plain, login

smtp_tls_wrappermode = yes
smtp_tls_security_level = encrypt

# 发件人地址映射（重要：确保发件人地址与认证邮箱一致）
smtp_generic_maps = hash:/etc/postfix/generic
```
2.  QQ 邮件配置：
```conf
# 基本设置
myhostname = your-server-hostname  # 替换为您的服务器主机名
mydomain = your-domain.com         # 替换为您的域名
myorigin = $mydomain
relayhost = [smtp.qq.com]:465
# SASL认证设置
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_sasl_mechanism_filter = plain, login
# TLS设置
smtp_use_tls = yes
smtp_tls_security_level = encrypt
smtp_tls_wrappermode = yes
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt

# 发件人地址映射（重要：确保发件人地址与认证邮箱一致）
smtp_generic_maps = hash:/etc/postfix/generic
```

### 4.  创建 SASL 认证文件
#### 创建`/etc/postfix/sasl_passwd`文件：
```bash
sudo nano /etc/postfix/sasl_passwd
```
#### 添加以下内容（使用您的 QQ/163 邮箱和授权码）：
```conf
# qq邮箱
[smtp.qq.com]:465 your_email@qq.com:授权码  # 非登录密码！
# 163邮箱
#[smtp.163.com]:465 your_email@163.com:授权码  # 非登录密码！

```
#### 设置文件权限并生成数据库：
```bash
sudo chmod 600 /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd
```
### 5.  创建发件人地址映射文件
#### 创建 `/etc/postfix/generic` 文件：
```bash
sudo nano /etc/postfix/generic
```
#### 添加以下内容（将 your-server-hostname 替换为您的实际服务器主机名）：
```bash
root@your-server-hostname 1009963012@qq.com
@your-server-hostname 1009963012@qq.com

```
#### 生成数据库：
```bash
sudo postmap /etc/postfix/generic
```

## 配置 Fail2ban 使用 Postfix
#### 编辑 `/etc/fail2ban/jail.local` 文件：
```bash
sudo nano /etc/fail2ban/jail.local
```
#### 确保包含以下配置：
```ini
[DEFAULT]
destemail = your-admin@example.com  # 替换为接收警报的邮箱
sender = 1009963012@qq.com
mta = sendmail
action = %(action_mw)s

[sshd]
enabled = true
port = ssh
logpath = %(sshd_log)s
maxretry = 3
findtime = 600
bantime = 3600
```

## 重启服务
```bash
sudo systemctl restart postfix
sudo systemctl restart fail2ban
```

## 验证 Postfix 与 Fail2ban 的集成
#### 确保 Postfix 能够处理来自 Fail2ban 的邮件发送请求：
```bash
# 测试从命令行发送邮件（模拟Fail2ban的行为）
echo "Subject: Fail2ban Test Mail" | sendmail -f 1009963012@qq.com your-email@example.com
```

### 测试触发ban行为后发送邮件
```bash
# 模拟封禁并发送邮件
sudo fail2ban-client set sshd banip 192.168.1.100
```





