---
title: secure-crt
date: 2016-11-28 22:43:19
categories: tools
tags: securecrt
---

securecrt的使用。

<!-- more -->


# 访问Google
在家不能访问Google，公司可以；在家可以通过VPN访问公司服务器。  
假设公司服务器地址是192.168.1.2。  

1. 在securecrt中为192.168.1.2建立一个session，然后右键打开“session options” -- "Port Forwarding" -- "Add":  
2. name随便填  
3. Local块中，IP地址填127.0.0.1，Port随便填（比如8899）；
4. Remote块中， Hostname填远程主机IP地址，这里是192.168.1.2，Port填写192.168.1.2上SSH端口，这里是22，勾上Dynamic forwarding...  
5. 断开连接，重新连接192.168.1.2。  
6. 在本地使用 netstat -an | grep 8899就可以看到本地监听了8899端口。  
7. Safari配置：Safari偏好设置--高级--代理--更改设置--勾上SOCKS代理--ip填写127.0.0.1，端口填写8899





