---
title: Easy_rsa--ssl工具
date: 2025-10-09 13:06:38
categories:
- linux
- easy-rsa
tags:
- tool
- openssl
---

# Easy-Rsa工具使用
## 工具介绍
easy-rsa 是一个用于构建和管理 PKI CA 的 CLI 实用程序。通俗地说， 这意味着创建一个根证书颁发机构，并请求和签名 证书，包括中间 CA 和证书吊销列表 （CRL）。

##  安装Easy-rsa
### 安装方式一（压缩包）推荐版本可选且新
####    1. 下载压缩包
下载链接：https://github.com/OpenVPN/easy-rsa/releases
![下载链接](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009131727.png)
```bash
cd ~/Download
wget https://github.com/OpenVPN/easy-rsa/releases/download/v3.2.4/EasyRSA-3.2.4.tgz
tar xvf EasyRSA-3.2.4.tgz
```
####    2. 配置环境变量
```bash
mkdir -p ~/SoftWare/EasyRSA-3.2.4
mv ~/Download/EasyRSA-3.2.4 ~/SoftWare/
cd ~/SoftWare/EasyRSA-3.2.4
echo "export EASY_RSA_HOME=$(pwd)" >> ~/.bashrc
echo 'PATH=$PATH:$EASY_RSA_HOME' >> ~/.bashrc
source ~/.bashrc
```
####    3. 验证
```bash
easyrsa version
```
出现以下信息则说明安装成功。
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009133227.png)
### 安装方式二（命令行）
####    1. 更新软件源
```bash
sudo apt update
```
####    2. 安装Easy-Rsa
```bash
sudo apt install easy-rsa
```
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009160001.png)


##  配置Easy-Rsa
### 1. 提供相关信息
####    1. 编辑`vars`文件
```bash
cd ~/SoftWare/EasyRSA-3.2.4
cp vars.example vars
vim vars
# 修改以下内容
set_var EASYRSA_DN      "org"

# Organizational fields (used with "org" mode and ignored in "cn_only" mode).
# These are the default values for fields which will be placed in the
# certificate.  Do not leave any of these fields blank, although interactively
# you may omit any specific field by typing the "." symbol (not valid for
# email).
#
# NOTE: The following characters are not supported
#       in these "Organizational fields" by Easy-RSA:
#       back-tick (`)
#
set_var EASYRSA_REQ_COUNTRY     "CN"
set_var EASYRSA_REQ_PROVINCE    "GUIZHOU"
set_var EASYRSA_REQ_CITY        "GuiYang"
set_var EASYRSA_REQ_ORG "feifei"
set_var EASYRSA_REQ_EMAIL       "1009963012@qq.com"
set_var EASYRSA_REQ_OU          "Tongji"


# 保存文件
:wq
```
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009162545.png)

### 2. 初始化
```bash
easyrsa init-pki
```
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009163027.png)

### 3. 复制`vars`到初始化目录
```bash
cp ~/SoftWare/EasyRSA-3.2.4/vars ~/pki/
```


##  生成CA根证书
```bash
easyrsa build-ca nopass  # nopass 表示根证书不加密
# 接下来的信息根据自己情况填写
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:fei-vitrue
```
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009163931.png)
自此CA根证书完成。

## 生成 Diffie-Hellman 参数
```bash
easyrsa gen-dh
# dh 文件生成较慢等待完成
```
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009164705.png)
dh.pem完成
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009165327.png)
## 不同场景的证书生成使用
### 1. nginx SSL证书生成
```bash
easyrsa --subject-alt-name="DNS:demo.com,DNS:www.demo.com,IP:192.168.0.1" build-server-full nginx nopass
# 下列为处理过程
Using Easy-RSA 'vars' configuration:
* /home/fei/pki/vars
Generating a RSA private key
...................................................................................................................................++++
..++++
writing new private key to '/home/fei/pki/bdc2b2a8/temp.04'
-----

Notice
------
Private-Key and Public-Certificate-Request files created.
Your files are:
* req: /home/fei/pki/reqs/nginx.req
* key: /home/fei/pki/private/nginx.key

You are about to sign the following certificate:

  Requested CN:     'nginx'
  Requested type:   'server'
  Valid for:        '825' days


subject=
    countryName               = CN
    stateOrProvinceName       = GUIZHOU
    localityName              = GuiYang
    organizationName          = feifei
    organizationalUnitName    = Tongji
    commonName                = nginx
    emailAddress              = 1009963012@qq.com

            X509v3 Subject Alternative Name:
                DNS:demo.com,DNS:www.demo.com,IP:192.168.0.1
# 这里输入yes确认上述信息
Type the word 'yes' to continue, or any other input to abort.
  Confirm requested details: yes

Using configuration from /home/fei/pki/2f1e0bf7/temp.02
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
countryName           :PRINTABLE:'CN'
stateOrProvinceName   :ASN.1 12:'GUIZHOU'
localityName          :ASN.1 12:'GuiYang'
organizationName      :ASN.1 12:'feifei'
organizationalUnitName:ASN.1 12:'Tongji'
commonName            :ASN.1 12:'nginx'
emailAddress          :IA5STRING:'1009963012@qq.com'
Certificate is to be certified until Jan 12 10:49:04 2028 GMT (825 days)

Write out database with 1 new entries
Data Base Updated

WARNING
=======
INCOMPLETE Inline file created:
* /home/fei/pki/inline/private/nginx.inline


Notice
------
Certificate created at:
* /home/fei/pki/issued/nginx.crt
# 结束
```
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009185159.png)
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009185229.png)

此时用于nginx的证书生成完成。
- 服务器证书：/home/fei/pki/issued/nginx.crt
- 服务器密钥：/home/fei/pki/private/nginx.key

示例：
将服务器证书和密钥复制到nginx的配置文件目录。
```bash
cp /home/fei/pki/issued/nginx.crt /path/nginx/conf
cp /home/fei/pki/private/nginx.key /path/nginx/conf
cp /home/fei/pki/dh.pem /path/nginx/conf

# nginx配置修改
vim /path/nginx/conf/nginx.conf
# nginx.conf

server {
    ssl_certificate nginx.crt;
    ssl_certificate_key nginx.key;
    ssl_dhparam dh.pem;

    # SSL 安全配置
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    
    # 其他配置
}
```
重启nginx即可。

补充：如果想要客户端中的https的不安全警告消失，则需要将`~/pki/ca.crt`文件安全的下载到客户端并添加的**受信任根证书**目录下。
![](https://blog.feizhufanfan.top:18088/minio/images/blog/20251009190456.png)
关闭浏览器重新打开即可。


### 2. openVPN场景
参考链接：[openVPN搭建](https://home.feizhufanfan.top:18088/2025/09/21/VPN%E5%AD%A6%E4%B9%A0-openVPN%E6%90%AD%E5%BB%BA/)



