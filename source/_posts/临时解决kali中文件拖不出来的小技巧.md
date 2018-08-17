---
title: 临时解决kali中文件拖不出来的小技巧
date: 2018-07-09 08:35:31
tags: 
	- Python
	- kali
	- 技巧
categories: 
	- Python
	- 技巧
---
用msf生成木马之后发现kali中的文件拖不出来，重新安装VMware tools又太麻烦，可以用这个方法解决下。
<!-- more -->
kali中自带python2、python3

直接在对应目录打开终端运行：
```
python3 -m http.server 8888

或者

python2 -m SimpleHTTPServer 8888
```
就可以运行起来静态服务。用来下载文件很方便。

然后本机访问http://kali的内网ip:8888