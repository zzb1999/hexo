---
title: 利用reGeorg+proxifier连接远程内网服务器
date: 2018-09-18 09:46:26
tags:
	- 渗透测试
	- 内网
	- reGeorg
	- proxifier
categories: 渗透测试
keywords:
description:
---
今天搞一个站拿到system权限之后lcx转发不出来，于是有了这篇文章。<!-- more -->

aspx的站已经拿到shell可以上传文件，直接上传对应脚本的reGeorg客户端，访问如下图代表正常运行。

![](http://image.ixysec.com/image/20180918/001.jpg)

然后开始设置Proxifier，运行软件，配置文件–>代理服务器

![](http://image.ixysec.com/image/20180918/002.jpg)

地址填写127.0.0.1，端口写一个未占用的端口，协议选择SOCKS5，填写完成点确定

![](http://image.ixysec.com/image/20180918/003.jpg)

![](http://image.ixysec.com/image/20180918/004.jpg)

再点击配置文件–>代理规则，选择添加一项规则

![](http://image.ixysec.com/image/20180918/005.jpg)

名称随便写，应用程序那里，选择python，目标主机和目标端口都任意，动作选Direct，然后点确定

![](http://image.ixysec.com/image/20180918/006.jpg)

如下图这样就可以了

![](http://image.ixysec.com/image/20180918/007.jpg)

运行reGeorg监听6666端口

`python reGeorgSocksProxy.py -p 6666 -u http://xxx.xxx.xxx/tunnel.aspx`

然后选择mstsc–>右键–>Proxifier–>proxy SOCKS5 127.0.0.1

![](http://image.ixysec.com/image/20180918/008.jpg)

就可以直接填写服务器的内网ip进行连接了。

![](http://image.ixysec.com/image/20180918/009.jpg)


工具下载：

reGeorg下载地址：https://github.com/sensepost/reGeorg

proxifier汉化版下载地址：http://forspeed.onlinedown.net/down/84267_20170616155632.zip


win10在连接的时候可能会出现什么Oracle验证错误，百度一下解决方法就可以轻松解决了。

[原文地址](https://assassins-white.github.io/2018/08/19/reGeorg+proxifier%E5%AE%9E%E7%8E%B0%E5%86%85%E7%BD%91%E7%A9%BF%E9%80%8F%E6%8B%BF%E6%9C%8D%E5%8A%A1%E5%99%A8/)