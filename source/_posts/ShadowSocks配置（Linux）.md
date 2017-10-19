---
title: ShadowSocks配置（Linux）
date: 2017-02-03 20:54:07
categories:
tags:
---
# 参考教程：

**强烈推荐教程[教程](http://www.auooo.com/2015/06/26/shadowsocks%EF%BC%88%E5%BD%B1%E6%A2%AD%EF%BC%89%E4%B8%8D%E5%AE%8C%E5%85%A8%E6%8C%87%E5%8D%97/#comment-4188)！！！**
========================================================================


----------
# 服务端配置：

Linux 直接 ssh 服务器：

ssh [ 用户名]@[IP]

*例如：
```
ssh root@45.76.206.109
```


----------


安装ShadowSocks：
使用Python原版，Ubuntu下直接输入：

```
apt install python-setuptools && easy_install pip
pip install shadowsocks
```
用来安装Python的组件和shadowsocks。


----------


ShadowSocks的配置：
创建配置文件：

```
vim /etc/shadowsocks.json
```

```
{
    "server":"your_server_ip",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"auooo.com",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```


----------


参数说明：

名称 |说明
--------|-------------
server 	|填入你的服务器 IP ，即当前操作的 VPS 的 IP 地址，必须修改
server_port| 	服务器端口，可以根据实际需要修改，或者保持默认
local_address| 	本地监听地址，建议保持默认
local_port 	|本地端口，这个参数一般保持默认即可
password 	|用来加密的密码，可以根据实际需要修改
timeout 	|单位秒，一般保持默认即可
method 	|默认的是”aes-256-cfb”，一般保持默认即可
fast_open 	|使用TCP_FASTOPEN, 参数选项true / false，一般保持默认即可
workers 	|worker的数量, 在 Unix/Linux 上有效，一般不用加此项


----------


启动SS服务：
```
nohup ssserver -c shadowsocks.json start &
```


----------
#客户端配置：
使用ShadowSocks-qt：
[官网下载&Wiki](https://github.com/shadowsocks/shadowsocks-qt5)


----------
配置客户端：
按照服务器的配置文件配置即可。

