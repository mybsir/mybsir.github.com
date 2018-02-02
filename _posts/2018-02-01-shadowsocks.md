---
layout: post
title:  Shadowsocks/privoxy/docker 科学上网
categories: k8s
description: Shadowsocks/privoxy/docker 科学上网
keywords: shadowsocks,ss,科学上网,docker 科学上网
---

# Shadowsocks/privoxy/docker 科学上网

## 一、shadowsocks是什么？
shadowsocks是一种基于Socks5代理方式的网络数据加密传输包，并采用Apache许可证、GPL、MIT许可证等多种自由软件许可协议开放源代码。shadowsocks分为服务器端和客户端，在使用之前，需要先将服务器端部署到服务器上面，然后通过客户端连接并创建本地代理。目前包使用Python、C、C++、C#、Go语言等编程语言开发。

运行原理：

Shadowsocks的运行原理与其他代理工具基本相同，使用特定的中转服务器完成数据传输。在服务器端部署完成后，用户需要按照指定的密码、加密方式和端口使用客户端软件与其连接。在成功连接到服务器后，客户端会在用户的电脑上构建一个本地Socks5代理。浏览网络时，网络流量会被分到本地socks5代理，客户端将其加密之后发送到服务器，服务器以同样的加密方式将流量回传给客户端，以此实现代理上网。

## 二、Shadowsocks部署（服务端）

### 2.1 环境介绍
```
阿里云ECS机器（香港或者其他国外机器）
系统：Centos 6
```

### 2.2 安装搭建

```
wget --no-check-certificate -O shadowsocks-all.sh https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks-all.sh
chmod +x shadowsocks-all.sh
./shadowsocks-all.sh 2>&1 | tee shadowsocks-all.log

#执行后会提示进行选择安装语言、访问密码、访问端口、加密方式。
笔者选择如下：
语言选择：Go
端口选择：10086
加密方式：aes-256-cfb
```
部署完毕后会有如下提示
```
Congratulations, Shadowsocks-Go server install completed!
Your Server IP        :  XX.XX.XX.XX
Your Server Port      :  10086
Your Password         :  123456
Your Encryption Method:  aes-256-cfb

Your QR Code: (For Shadowsocks Windows, OSX, Android and iOS clients)
 ss://YWVzLTI1Ni1jZmI6S2liZXkuYzBtLjIwMUJANDcuOTEuMTk2LjEwMzoxMDA4Ng==
Your QR Code has been saved as a PNG file path:
 /root/shadowsocks_go_qr.png

Welcome to visit: https://teddysun.com/486.html
Enjoy it!
```
### 2.3 开放端口或者阿里云安全组配置（如果有需要）

```
iptables -A INPUT -p tcp -m state --state NEW -m tcp --dport 8989 -j ACCEPT
service iptables save
service iptables restart
```

### 2.4 卸载方法

```
./shadowsocks-all.sh uninstall
```

## 三、客户端使用

### 3.1 MAC客户端

```
下载地址：https://github.com/shadowsocks/shadowsocks-iOS/releases
```

![](/images/posts/k8s/mac_shadowsocks_1.png)
![](/images/posts/k8s/mac_shadowsocks_2.png)

### 3.2 Centos代理

#### 3.2.1 shadowsocks安装与设置

```
yum -y install epel-release
yum -y install python-pip
pip install shadowsocks
```

#### 3.2.2 配置shadowsocks客户端

```
vi /etc/shadowsocks.json

{
    "server":"SERVER-IP",           # 你的服务器ip
    "server_port":PORT,             # 服务器端口
    "local_address": "127.0.0.1",   # 若想其它机器可访问，设置为0.0.0.0
    "local_port":1080,              # sock端口
    "password":"PASSWORD",          # 密码
    "timeout":300,
    "method":"aes-256-cfb"          # 加密方式
}

```

#### 3.2.3 编写shadowsocks服务（CENTOS 7）

```
vi /etc/systemd/system/shadowsocks.service


[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/sslocal -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target


# 启动服务
systemctl start shadowsocks
systemctl enable shadowsocks
```

#### 3.2.4 启动shadowsocks服务（CENTOS 6）

```
nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &
```


#### 3.2.5 安装Privoxy

> 安装好了`shadowsocks`，但是我们在shell里执行的命令，发起的网络请求并不支持`sock5`代理，只支持`http/https`代理。为此我们需要安装`privoxy`代理，把电脑上的所有HTTP请求转发给`shadowsocks`

```
yum -y install privoxy
```

#### 3.2.6 配置privoxy

```
echo 'forward-socks5 / 127.0.0.1:1080 .' >> /etc/privoxy/config
```

#### 3.2.7 启动并保持开机自启

```
systemctl start privoxy
systemctl enable privoxy
```

### Docker代理设置

```
vi /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://127.0.0.1:8118/" "NO_PROXY=localhost,127.0.0.1,gj7l88s1.mirror.aliyuncs.com,docker.io,registry.cn-hangzhou.aliyuncs.com,acs-cn-hangzhou-mirror.oss-cn-hangzhou.aliyuncs.com"


# NO_PROXY可让其他镜像继续使用阿里云加速。

```

```
#重启docker
systemctl daemon-reload
systemctl restart docker
```


#### 测试拉取镜像

```
[root@node2 ~]# docker pull  gcr.io/google_containers/kubernetes-dashboard-amd64:v1.8.0
v1.8.0: Pulling from google_containers/kubernetes-dashboard-amd64
39e01bcdd352: Pull complete
Digest: sha256:71a0de5c6a21cb0c2fbcad71a4fef47acd3e61cd78109822d35e1742f9d8140d
Status: Downloaded newer image for gcr.io/google_containers/kubernetes-dashboard-amd64:v1.8.0
```

参考：

(http://www.wisely.top/2017/05/15/shadowsocks-privoxy-docker-centos-wall/)

(http://www.cnblogs.com/tianhei/p/7428622.html)

(https://docs.lvrui.io/2016/12/12/Linux%E4%B8%AD%E4%BD%BF%E7%94%A8ShadowSocks-Privoxy%E4%BB%A3%E7%90%86/)


**完结**