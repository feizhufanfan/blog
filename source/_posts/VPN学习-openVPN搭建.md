---
title: VPN学习--openVPN搭建
date: 2025-09-21 22:09:59
categories:
- linux
- openVPN
tags:
- linux
- 网络安全
- OpenVPN
---

#   VPN--OpenVPN搭建
##  依赖环境
### 1.  安装依赖环境包
#### 安装 **OpenVPN**  
```bash
sudo apt update
sudo apt install openvpn
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921224525.png)

#### 安装 **[easy-rsa](https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz)**
1.  下载`Easy-rsa`包
```bash
mkdir -p ~/Download && cd ~/Download
wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
```
![下载Easy-rsa](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921225147.png)

1.  将`Easy-rsa`添加到环境变量
```bash
tar -xvf EasyRSA-3.2.4.tgz
mkdir -p ~/SoftWare 
mv EasyRSA-3.2.4/ ~/SoftWare/
cd ~/SoftWare/EasyRSA-3.2.4
echo "export EASYRSA_HOME=$(pwd)" >> ~/.bashrc
echo 'export PATH=$PATH:$EASYRSA_HOME' >> ~/.bashrc
source ~/.bashrc
```
![解压Esay包](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921230008.png)
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921230646.png)
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921230748.png)

出现以上结果说明添加到环境变量成功。

##  配置OpenVPN
### 1.  设置 Easy-RSA 和构建证书颁发机构 (CA)
#### 1. 初始化 PKI
- 生成PKI目录
```bash
cd ~
easyrsa init-pki
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921231237.png)
- 填写证书信息
```bash
cp pki/vars.example pki/vars
vim pki/vars 
# 找到一下内容并修改
set_var EASYRSA_DN      "org"

set_var EASYRSA_REQ_COUNTRY     "CN"
set_var EASYRSA_REQ_PROVINCE    "GuiZhou"
set_var EASYRSA_REQ_CITY        "GuiYang"
set_var EASYRSA_REQ_ORG "feifei"
set_var EASYRSA_REQ_EMAIL       "1009963012@qq.com"
set_var EASYRSA_REQ_OU          "Tongji"

#保存内容
:wq
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921232002.png)

再次执行初始化命令确认配置文件正确导入
```bash
easyrsa init-pki
no
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921232203.png)

#### 2. 构建CA  
在执行此命令时，系统会提示您输入 CA 的通用名称（Common Name）。您可以直接按 Enter 键接受默认名称也可以根据自身情况修改名称。
```bash
easyrsa build-ca nopass
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921232847.png)

此命令会创建两个重要文件：pki/ca.crt (CA 公共证书) 和 pki/private/ca.key (CA 私钥)。

### 2.  生成服务器证书和密钥
#### 1. 生成服务器证书请求和私钥
将 server 替换为您喜欢的服务器名称。nopass 选项表示不加密私钥。
```bash
easyrsa gen-req server nopass
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921233328.png)
同样，系统会提示输入通用名称，可以直接按 Enter 键。

#### 2. 签署服务器证书：
使用我们刚刚创建的 CA 来签署服务器证书请求。
```bash
easyrsa sign-req server server
```
在提示确认时，输入 yes 并按 Enter。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250921233719.png)
初始生成的时候可能会应为权限问题导致一个文件生成失败，我们手动创建一个。
```bash
# 我们将之前生成的服务器证书删除重新生成
rm pki/issued/server.crt
# 创建好文件后再次执行
easyrsa sign-req server server
yes
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922001014.png)

#### 3. 生成 Diffie-Hellman 参数
这用于密钥交换。
```bash
easyrsa gen-dh
```
这里的密钥生成需要花费一些时间，等待完成即可
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922001313.png)
密钥生成完成。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922001523.png)

#### 4. 生成 TLS 认证密钥
这可以增加额外的安全层，以防止 DoS 攻击。
```bash
openvpn --genkey --secret ta.key

```
#### 5. 将所需文件复制到 OpenVPN 目录：
```bash
sudo cp pki/ca.crt /etc/openvpn/
sudo cp pki/issued/server.crt /etc/openvpn/
sudo cp pki/private/server.key /etc/openvpn/
sudo cp pki/dh.pem /etc/openvpn/
sudo cp ta.key /etc/openvpn/
```
### 3.  生成客户端证书和密钥
现在，为每个需要连接 VPN 的用户生成一个客户端证书和密钥
#### 1. 生成客户端证书请求和私钥：
将 client1 替换为实际的用户名。为每个用户重复此过程。
```bash
easyrsa gen-req client1 nopass
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922002443.png)

#### 2. 签署客户端证书：

```bash
easyrsa sign-req client client1
yes
```
同样，输入 yes 进行确认。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922002654.png)
***重要提示：*** 您需要将以下文件安全地分发给客户端：
- `pki/ca.crt`
- `pki/issued/client1.crt`
- `pki/private/client1.key`

### 4.  配置 OpenVPN 服务
现在我们来创建 OpenVPN 服务器的配置文件。
#### 1. 创建并编辑`server.conf`文件：
复制示例配置文件并进行修改。
```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
sudo gzip -d /etc/openvpn/server.conf.gz
sudo vim /etc/openvpn/server.conf
```
#### 2. 修改以下配置项：
```bash
local 0.0.0.0
port 1149
proto udp
dev tun

ca ca.crt
cert server.crt
key server.key

# Diffie hellman parameters.
# Generate your own with:
#   openssl dhparam -out dh2048.pem 2048
dh dh.pem

# ethernet bridging. See the man page for more info.
# 以下为设置VPN分配给客户端的网段
server 172.168.0.0 255.255.255.0
status /var/log/openvpn/openvpn-status.log
log         /var/log/openvpn/openvpn.log
log-append  /var/log/openvpn/openvpn.log
ifconfig-pool-persist /var/log/openvpn/ipp.txt

push "route 192.168.0.0 255.255.255.0"   # 只推送VPN服务所在网段的路由表给客户端
;push "redirect-gateway def1 bypass-dhcp" # 表示所有客户端的流量全部经过VPN。
;push "dhcp-option DNS 8.8.8.8" # 推送给客户端DNS
;push "dhcp-option DNS 8.8.4.4"



tls-auth ta.key 0
# 在客户端配置中添加
tls-version-min 1.2

cipher AES-256-CBC
auth SHA256

comp-lzo    # 表示启动数据压缩节省带宽

persist-key
persist-tun

verb 3
mute 20
explicit-exit-notify 1
# 启用用户名密码验证脚本
auth-user-pass-verify /etc/openvpn/checkpsw.sh via-env
# 不使用`client-cert-not-required`，表示需要证书+密码双因素认证
# 如果启用`client-cert-not-required`，则代表只使用用户名密码认证（安全性较低，不推荐）
username-as-common-name
script-security 3
```
#### 3. 创建用户名密码验证脚本和凭据文件
`sudo vim /etc/openvpn/checkpsw.sh`
```bash
#!/bin/bash
###########################################################
# checkpsw.sh (C) 2004 Mathias Sundman <mathias@openvpn.se>
# 该脚本通过明文文件验证OpenVPN用户
# 密码文件每行应包含一个用户：用户名 密码
###########################################################

PASSFILE="/etc/openvpn/psw-file"
LOG_FILE="/etc/openvpn/openvpn-password.log"
TIME_STAMP=$(date "+%Y-%m-%d %T")

if [ ! -r "${PASSFILE}" ]; then
  echo "${TIME_STAMP}: Could not open password file \"${PASSFILE}\" for reading." >> ${LOG_FILE}
  exit 1
fi

CORRECT_PASSWORD=$(awk '!/^;/&&!/^#/&&$1=="'${username}'"{print $2;exit}' ${PASSFILE})

if [ "${CORRECT_PASSWORD}" = "" ]; then
  echo "${TIME_STAMP}: User does not exist: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
  exit 1
fi

if [ "${password}" = "${CORRECT_PASSWORD}" ]; then
  echo "${TIME_STAMP}: Successful authentication: username=\"${username}\"." >> ${LOG_FILE}
  exit 0
fi

echo "${TIME_STAMP}: Incorrect password: username=\"${username}\", password=\"${password}\"." >> ${LOG_FILE}
exit 1
```
保存退出后赋予运行权限
`sudo chmod +x /etc/openvpn/checkpsw.sh`

#### 4. 创建用户名密码文件`psw-file`
```bash
sudo vim /etc/openvpn/psw-file
# 每行定义一个用户，格式为 用户名 密码：
user1 123456
user2 abcd1234
your_username your_secure_password
```
保存退出后赋予root只读权限
`sudo chmod 0400 /etc/openvpn/psw-file`














##  配置网络防火墙
### 1.  启用IP转发
编辑 /etc/sysctl.conf 文件，取消注释或添加以下行。
```bash
net.ipv4.ip_forward=1
```
应用更改：
```bash
sudo sysctl -p
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922112748.png)

### 2.  配置防火墙（如果使用ufw）
- 允许OpenVPN端口（例如默认的1194 UDP）
```bash
sudo ufw allow 1194/udp
```
- 配置NAT规则，使客户端能通过VPN服务器访问互联网（假设你的服务器公网网卡是 eth0）
```bash
# 在/etc/ufw/before.rules文件开头添加（在`*filter`节之前）
sudo vim /etc/ufw/before.rules
```
添加：
```text
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
COMMIT
```
- 还需修改 /etc/default/ufw，将 DEFAULT_FORWARD_POLICY 从 DROP 改为 ACCEPT
```bash
DEFAULT_FORWARD_POLICY="ACCEPT"
```
- 重新加载UFW以使更改生效：
```bash
sudo ufw disable
sudo ufw enable
```
- 如果使用 iptables 直接管理，添加相应的NAT和允许规则。




##  启动OpenVPN服务并测试
### 1.  启动OpenVPN服务：
```bash
sudo systemctl start openvpn.service    # `server`对应/etc/openvpn/server.conf
```
### 2.  设置开机自启：
```bash
sudo systemctl enable openvpn.service
```

### 3.  检查服务状态：
```bash
sudo systemctl status openvpn.service
```
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922130549.png)

### 4.  查看日志是否有错误
```bash
sudo tail -f /var/log/openvpn.log
```
## 生成客户端配置文件
### 1.  为客户端的配置文件（.ovpn文件）准备目录和文件
#### 1. 创建`OpenVPN`文件夹
```bash
mkdir -p ~/OpenVPN && cd ~/OpenVPN
```
#### 2. 复制证书及密钥
需要的文件
- `ca.crt` (CA证书)
- `client1.crt` (客户端证书)
- `client1.key` (客户端私钥)
- `ta.key`  (如果服务器配置使用了TLS认证)
```bash
cp ~/pki/ca.crt ./
cp ~/ta.key ./
cp ~/pki/issued/client1.crt ./

```

#### 3. 创建客户端基本配置文件`base.conf`
客户端基本配置文件：
`vim base.conf`
```bash
# 编辑内容如下
client
dev tun
proto udp
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote 192.168.0.10 1149  # 这里替换实际的VPN服务器的IP地址或者域名后面为端口
resolv-retry infinite
nobind
persist-key
persist-tun
auth-user-pass
auth-nocache
remote-cert-tls server
key-direction 1
cipher AES-256-CBC
auth SHA256
comp-lzo

# Set log file verbosity.
verb 3

# Silence repeating messages
mute 20
```

#### 4. 创建配置文件生成脚本`generateConfig.sh`
`vim generateConfig.sh`
```bash
#!/bin/bash
# First argument: Client identifier
KEY_DIR=$(pwd)
OUTPUT_DIR=$(pwd)
BASE_CONFIG=$(pwd)/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```
保存退出后赋予运行权限
`chmod +x generateConfig.sh`

#### 5. 生成客户端配置文件。
确认目录下文件是否都准备就绪。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922124936.png)
执行命令
```bash
sh ./generateConfig.sh client1
```
目录下生成`client1.ovpn`
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922125128.png)


## 测试OpenVPN服务
### 下载安装包[openvpn-connect-v3-windows](https://openvpn.net/downloads/openvpn-connect-v3-windows.msi)
采用默认安装等待安装完成。
从服务器下载`client1.ovpn`到本地。
### 导入配置文件
将文件拖入或者选择位置导入。
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922130749.png)
导入成功后
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922131455.png)
输入之前创建的用户名
点击connect
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922131543.png)
输入用户密码
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922142256.png)
![](http://blog.feizhufanfan.top:18088/minio/images/blog/20250922141907.png)








