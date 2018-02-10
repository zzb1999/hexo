---
title: Mysql
date: 2018-02-04 14:15:05
tags:
	- Mysql
	- Python
	- Database
categories: Database
---
Mysql数据库，是当前应用非常广泛的一款关系型数据库
<!-- more -->
查看[官方网站](https://www.mysql.com/)
查看[数据库排名](https://db-engines.com/en/ranking)

## 简介与安装
### 数据库简介
+ 人类在进化的过程中，创造了数字、文字、符号等来进行数据的记录，但是承受着认知能力和创造能力的提升，数据量越来越大，对于数据的记录和准确查找，成为了一个重大难题
+ 计算机诞生后，数据开始在计算机中存储并计算，并设计出了数据库系统
+ 数据库系统解决的问题：持久化存储，优化读写，保证数据的有效性
+ 当前使用的数据库，主要分为两类
    + 文档型，如sqlite，就是一个文件，通过对文件的复制完成数据库的复制
    + 服务型，如mysql、postgre，数据存储在一个物理文件中，但是需要使用终端以tcp/ip协议连接，进行数据库的读写操作

#### E-R模型
+ 当前物理的数据库都是按照E-R模型进行设计的
+ E表示entry，实体
+ R表示relationship，关系
+ 一个实体转换为数据库中的一个表
+ 关系描述两个实体之间的对应规则，包括
    + 一对一
    + 一对多
    + 多对多
+ 关系转换为数据库表中的一个列 *在关系型数据库中一行就是一个对象

#### 三范式
+ 经过研究和对使用中问题的总结，对于设计数据库提出了一些规范，这些规范被称为范式
+ 第一范式（1NF)：列不可拆分
+ 第二范式（2NF)：唯一标识
+ 第三范式（3NF)：引用主键
+ 说明：后一个范式，都是在前一个范式的基础上建立的

### 安装
+ 安装
```
sudo apt-get install mysql-server mysql-client
然后按照提示输入
```
#### 管理服务
+ 启动
```
service mysql start
```
+ 停止
```
service mysql stop
```
+ 重启
```
service mysql restart
```
#### 允许远程连接
+ 找到mysql配置文件并修改
```
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
将bind-address=127.0.0.1注释
```
+ 登录mysql，运行命令
```
grant all privileges on *.* to 'root'@'%' identified by 'mysql' with grant option;
flush privileges;
```
+ 重启mysql

### 数据完整性
+ 一个数据库就是一个完整的业务单元，可以包含多张表，数据被存储在表中
+ 在表中为了更加准确的存储数据，保证数据的正确有效，可以在创建表的时候，为表添加一些强制性的验证，包括数据字段的类型、约束

#### 字段类型
+ 在mysql中包含的数据类型很多，这里主要列出来常用的几种
+ 数字：int,decimal
+ 字符串：varchar,text
+ 日期：datetime
+ 布尔：bit

#### 约束
+ 主键primary key
+ 非空not null
+ 惟一unique
+ 默认default
+ 外键foreign key

#### 逻辑删除
+ 对于重要数据，并不希望物理删除，一旦删除，数据无法找回
+ 一般对于重要数据，会设置一个isDelete的列，类型为bit，表示逻辑删除
+ 大于大量增长的非重要数据，可以进行物理删除
+ 数据的重要性，要根据实际开发决定

### 命令脚本操作
#### 使用命令连接
+ 打开终端，运行命令
```
mysql -uroot -proot -h127.0.0.1
-u root
-p 安装时设置的密码
-h 连接的主机ip地址
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/mysql1.jpg)

+ 退出登录
```
quit或exit
```
+ 登录成功后，输入如下命令查看效果
```
查看版本：select version();
显示当前时间：select now();
```
+ 注意：在语句结尾要使用分号;

#### 数据库操作
+ 查看所有数据库
```
show databases;
```
+ 创建数据库
```
create database 数据库名 charset=utf8;
```
+ 删除数据库
```
drop database 数据库名;
```
+ 切换当前数据库
```
use 数据库名;
```
+ 查看当前选择的数据库
```
select database();
```
#### 表操作
+ 查看当前数据库中所有表
```
show tables;
```
+ 创建表
```
create table 表名(列...);
列的格式：列的名称 类型 约束
create table student(
    id int not null primary key auto_increment,
    name varchar(10) not null,
    gender bit default 1,
    birthday datetime,
    isDelete bit default 0
     );
auto_increment表示自动增长
```
+ 修改表
```
alter table 表名 add|change|drop 列名 类型;
如：
alter table students add birthday datetime;
```
+ 删除表
```
drop table 表名;
```
+ 查看表结构
```
desc 表名;
```
+ 更改表名称
```
rename table 原表名 to 新表名;
```
+ 查看表的创建语句
```
show create table '表名';
```
#### 数据操作
+ 查询
```
select * from 表名
```
+ 增加
```
全列插入：insert into 表名 values(...);
缺省插入：insert into 表名(列1,...) values(值1,...);
同时插入多条数据：insert into 表名 values(...),(...)...;
或insert into 表名(列1,...) values(值1,...),(值1,...)...;
```
+ 主键列是自动增长，但是在全列插入时需要占位，通常使用0，插入成功后以实际数据为准
+ 修改
```
update 表名 set 列1=值1,... where 条件
```
+ 删除
```
delete from 表名 where 条件
```
+ 逻辑删除，本质就是修改操作update
```
alter table students add isdelete bit default 0;
如果需要删除则
update students isdelete=1 where ...;
```
#### 备份与恢复
##### 数据备份
+ 进入超级管理员
```
sudo -s
```
+ 进入mysql库目录
```
cd /var/lib/mysql
```
+ 运行mysqldump命令
```
mysqldump –uroot –p 数据库名 > ~/Desktop/备份文件.sql;
按提示输入mysql的密码
```
##### 数据恢复
+ 连接mysql，创建数据库
+ 退出连接，执行如下命令
```
mysql -uroot –p 数据库名 < ~/Desktop/备份文件.sql
根据提示输入mysql密码
```

## 查询
### 查询的基本语法
+ select * from 表名;
+ from关键字后面写表名，表示数据来源于是这张表
+ select后面写表中的列名，如果是*表示在结果中显示表中所有列
+ 在select后面的列名部分，可以使用as为列起别名，这个别名出现在结果集中
+ 如果要查询多个列，之间使用逗号分隔

#### 消除重复行
+ 在select后面列前使用distinct可以消除重复的行
```
select distinct gender from students;
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/distinct.jpg)

### 条件
+ 使用where子句对表中的数据筛选，结果为true的行会出现在结果集中
+ 语法如下：
```
select * from 表名 where 条件;
```
#### 比较运算符
+ 等于=
+ 大于>
+ 大于等于>=
+ 小于<
+ 小于等于<=
+ 不等于!=或<>
+ 查询编号大于3的学生
```
select * from students where id>3;
```
+ 查询编号不大于4的科目
```
select * from subjects where id<=4;
```
+ 查询姓名不是“黄蓉”的学生
```
select * from students where sname!='黄蓉';
```
+ 查询没被删除的学生
```
select * from students where isdelete=0;
```
#### 逻辑运算符
+ and
+ or
+ not
+ 查询编号大于3的女同学
```
select * from students where id>3 and gender=0;
```
+ 查询编号小于4或没被删除的学生
```
select * from students where id<4 or isdelete=0;
```
#### 模糊查询like
+ %表示任意多个任意字符
+ _表示一个任意字符
+ 查询姓黄的学生
```
select * from students where sname like '黄%';
```
+ 查询姓黄并且名字是一个字的学生
```
select * from students where sname like '黄_';
```
+ 查询姓黄或叫靖的学生
```
select * from students where sname like '黄%' or sname like '%靖%';
```
#### 范围查询
+ in表示在一个非连续的范围内
+ 查询编号是1或3或8的学生
```
select * from students where id in(1,3,8);
```
+ between ... and ...表示在一个连续的范围内
+ 查询学生是3至8的学生
```
select * from students where id between 3 and 8;
```
+ 查询学生是3至8的男生
```
select * from students where id between 3 and 8 and gender=1;
```
#### 空判断
+ 注意：null与''是不同的
+ 判空is null
+ 查询没有填写地址的学生
```
select * from students where hometown is null;
```
+ 判非空is not null
+ 查询填写了地址的学生
```
select * from students where hometown is not null;
```
+ 查询填写了地址的女生
```
select * from students where hometown is not null and gender=0;
```
#### 优先级
+ 小括号，not，比较运算符，逻辑运算符
+ and比or先运算，如果同时出现并希望先算or，需要结合()使用

### 聚合
+ 为了快速得到统计数据，提供了5个聚合函数
+ count(*)表示计算总行数，括号中写星与列名，结果是相同的
+ 查询学生总数
```
select count(*) from students;
```
+ max(列)表示求此列的最大值
+ 查询女生的编号最大值
```
select max(id) from students where gender=0;
```
+ min(列)表示求此列的最小值
+ 查询未删除的学生最小编号
```
select min(id) from students where isdelete=0;
```
+ sum(列)表示求此列的和
+ 查询男生的编号之和
```
select sum(id) from students where gender=1;
```
+ avg(列)表示求此列的平均值
+ 查询未删除女生的编号平均值
```
select avg(id) from students where isdelete=0 and gender=0;
```
### 分组
+ 按照字段分组，表示此字段相同的数据会被放到一个组中
+ 分组后，只能查询出相同的数据列，对于有差异的数据列无法出现在结果集中
+ 可以对分组后的数据进行统计，做聚合运算
+ 语法：
```
select 列1,列2,聚合... from 表名 group by 列1,列2,列3...
```
+ 查询男女生总数
```
select gender as 性别,count(*)
from students
group by gender;
```
+ 查询各城市人数
```
select hometown as 家乡,count(*)
from students
group by hometown;
```
#### 分组后的数据筛选
+ 语法：
```
select 列1,列2,聚合... from 表名
group by 列1,列2,列3...
having 列1,...聚合...
```
+ having后面的条件运算符与where的相同
+ 查询男生总人数
```
方案一
select count(*)
from students
where gender=1;
-----------------------------------
方案二：
select gender as 性别,count(*)
from students
group by gender
having gender=1;
```
#### 对比where与having
+ where是对from后面指定的表进行数据筛选，属于对原始数据的筛选
+ having是对group by的结果进行筛选

### 排序
+ 为了方便查看数据，可以对数据进行排序
+ 语法：
```
select * from 表名
order by 列1 asc|desc,列2 asc|desc,...
```
+ 将行数据按照列1进行排序，如果某些行列1的值相同时，则按照列2排序，以此类推
+ 默认按照列值从小到大排列
+ asc从小到大排列，即升序
+ desc从大到小排序，即降序
+ 查询未删除男生学生信息，按学号降序
```
select * from students
where gender=1 and isdelete=0
order by id desc;
```
+ 查询未删除科目信息，按名称升序
```
select * from subject
where isdelete=0
order by stitle;
```
### 分页
#### 获取部分行
+ 当数据量过大时，在一页中查看数据是一件非常麻烦的事情
+ 语法
```
select * from 表名
limit start,count
```
+ 从start开始，获取count条数据
+ start索引从0开始

#### 示例：分页
+ 已知：每页显示m条数据，当前显示第n页
+ 求总页数：此段逻辑后面会在python中实现
    + 查询总条数p1
    + 使用p1除以m得到p2
    + 如果整除则p2为总数页
    + 如果不整除则p2+1为总页数
+ 求第n页的数据
```
select * from students
where isdelete=0
limit (n-1)*m,m
```
### 总结
+ 完整的select语句
```
select distinct *
from 表名
where ....
group by ... having ...
order by ...
limit star,count
```
+ 执行顺序为：
    + from 表名
    + where ....
    + group by ...
    + select distinct *
    + having ...
    + order by ...
    + limit star,count
+ 实际使用中，只是语句中某些部分的组合，而不是全部

## 高级
### 外键
+ 思考：怎么保证关系列数据的有效性呢？任何整数都可以吗？
+ 答：必须是学生表中id列存在的数据，可以通过外键约束进行数据的有效性验证
+ 为stuid添加外键约束
```
alter table scores add constraint stu_sco foreign key(stuid) references students(id);
```
+ 此时插入或者修改数据时，如果stuid的值在students表中不存在则会报错
+ 在创建表时可以直接创建约束
```
create table scores(
id int primary key auto_increment,
stuid int,
subid int,
score decimal(5,2),
foreign key(stuid) references students(id),
foreign key(subid) references subjects(id)
);
```
#### 外键的级联操作
+ 在删除students表的数据时，如果这个id值在scores中已经存在，则会抛异常
+ 推荐使用逻辑删除，还可以解决这个问题
+ 可以创建表时指定级联操作，也可以在创建表后再修改外键的级联操作
+ 语法
```
alter table scores add constraint stu_sco foreign key(stuid) references students(id) on delete cascade;
```
+ 级联操作的类型包括：
    + restrict（限制）：默认值，抛异常
    + cascade（级联）：如果主表的记录删掉，则从表中相关联的记录都将被删除
    + set null：将外键设置为空
    + no action：什么都不做

### 连接
+ 问：查询每个学生每个科目的分数
+ 分析：学生姓名来源于students表，科目名称来源于subjects，分数来源于scores表，怎么将3个表放到一起查询，并将结果显示在同一个结果集中呢？
+ 答：当查询结果来源于多张表时，需要使用连接查询
+ 关键：找到表间的关系，当前的关系是
    + students表的id---scores表的stuid
    + subjects表的id---scores表的subid
+ 则上面问题的答案是：
```
select students.sname,subjects.stitle,scores.score
from scores
inner join students on scores.stuid=students.id
inner join subjects on scores.subid=subjects.id;
```
+ 结论：当需要对有关系的多张表进行查询时，需要使用连接join

#### 连接查询
+ 连接查询分类如下：
    + 表A inner join 表B：表A与表B匹配的行会出现在结果中
    + 表A left join 表B：表A与表B匹配的行会出现在结果中，外加表A中独有的数据，未对应的数据使用null填充
    + 表A right join 表B：表A与表B匹配的行会出现在结果中，外加表B中独有的数据，未对应的数据使用null填充
+ 在查询或条件中推荐使用“表名.列名”的语法
+ 如果多个表中列名不重复可以省略“表名.”部分
+ 如果表的名称太长，可以在表名后面使用' as 简写名'或' 简写名'，为表起个临时的简写名称

#### 练习
+ 查询学生的姓名、平均分
```
select students.sname,avg(scores.score)
from scores
inner join students on scores.stuid=students.id
group by students.sname;
```
+ 查询男生的姓名、总分
```
select students.sname,avg(scores.score)
from scores
inner join students on scores.stuid=students.id
where students.gender=1
group by students.sname;
```
+ 查询科目的名称、平均分
```
select subjects.stitle,avg(scores.score)
from scores
inner join subjects on scores.subid=subjects.id
group by subjects.stitle;
```
+ 查询未删除科目的名称、最高分、平均分
```
select subjects.stitle,avg(scores.score),max(scores.score)
from scores
inner join subjects on scores.subid=subjects.id
where subjects.isdelete=0
group by subjects.stitle;
```

### 自关联
+ 设计省信息的表结构provinces
    + id
    + ptitle
+ 设计市信息的表结构citys
    + id
    + ctitle
    + proid
+ citys表的proid表示城市所属的省，对应着provinces表的id值
+ 问题：能不能将两个表合成一张表呢？
+ 思考：观察两张表发现，citys表比provinces表多一个列proid，其它列的类型都是一样的
+ 意义：存储的都是地区信息，而且每种信息的数据量有限，没必要增加一个新表，或者将来还要存储区、乡镇信息，都增加新表的开销太大
+ 答案：定义表areas，结构如下
    + id
    + ptitle
    + pid
+ 因为省没有所属的省份，所以可以填写为null
+ 城市所属的省份pid，填写省所对应的编号id
+ 这就是自关联，表中的某一列，关联了这个表中的另外一列，但是它们的业务逻辑含义是不一样的，城市信息的pid引用的是省信息的id
+ 在这个表中，结构不变，可以添加区县、乡镇街道、村社区等信息
+ 创建areas表的语句如下：
```
create table areas(
id int primary key,
atitle varchar(20),
pid int,
foreign key(pid) references areas(id)
);
```
+ 查询省的名称为“山西省”的所有城市
```
select city.* from areas as city
inner join areas as province on city.pid=province.id
where province.atitle='山西省';
```
+ 查询市的名称为“广州市”的所有区县
```
select dis.*,dis2.* from areas as dis
inner join areas as city on city.id=dis.pid
left join areas as dis2 on dis.id=dis2.pid
where city.atitle='广州市';
```

### 内置函数
#### 字符串函数
+ 查看字符的ascii码值ascii(str)，str是空串时返回0
```
select ascii('a');
```
+ 查看ascii码值对应的字符char(数字)
```
select char(97);
```
+ 拼接字符串concat(str1,str2...)
```
select concat(12,34,'ab');
```
+ 包含字符个数length(str)
```
select length('abc');
```
+ 截取字符串
    + left(str,len)返回字符串str的左端len个字符
    + right(str,len)返回字符串str的右端len个字符
    + substring(str,pos,len)返回字符串str的位置pos起len个字符
```
select substring('abc123',2,3);
```
+ 去除空格
    + ltrim(str)返回删除了左空格的字符串str
    + rtrim(str)返回删除了右空格的字符串str
    + trim([方向 remstr from str)返回从某侧删除remstr后的字符串str，方向词包括both、leading、trailing，表示两侧、左、右
```
select trim('  bar   ');
select trim(leading 'x' FROM 'xxxbarxxx');
select trim(both 'x' FROM 'xxxbarxxx');
select trim(trailing 'x' FROM 'xxxbarxxx');
```
+ 返回由n个空格字符组成的一个字符串space(n)
```
select space(10);
```
+ 替换字符串replace(str,from_str,to_str)
```
select replace('abc123','123','def');
```
+ 大小写转换，函数如下
    + lower(str)
    + upper(str)
```
select lower('aBcD');
```

#### 数学函数
+ 求绝对值abs(n)
```
select abs(-32);
```
+ 求m除以n的余数mod(m,n)，同运算符%
```
select mod(10,3);
select 10%3;
```
+ 地板floor(n)，表示不大于n的最大整数
```
select floor(2.3);
```
+ 天花板ceiling(n)，表示不小于n的最大整数
```
select ceiling(2.3);
```
+ 求四舍五入值round(n,d)，n表示原数，d表示小数位置，默认为0
```
select round(1.6);
```
+ 求x的y次幂pow(x,y)
```
select pow(2,3);
```
+ 获取圆周率PI()
```
select PI();
```
+ 随机数rand()，值为0-1.0的浮点数
```
select rand();
```
+ 还有其它很多三角函数，使用时可以查询文档

#### 日期时间函数
+ 获取子值，语法如下
    + year(date)返回date的年份(范围在1000到9999)
    + month(date)返回date中的月份数值
    + day(date)返回date中的日期数值
    + hour(time)返回time的小时数(范围是0到23)
    + minute(time)返回time的分钟数(范围是0到59)
    + second(time)返回time的秒数(范围是0到59)
```
select year('2016-12-21');
```
+ 日期计算，使用+-运算符，数字后面的关键字为year、month、day、hour、minute、second
```
select '2016-12-21'+interval 1 day;
```
+ 日期格式化date_format(date,format)，format参数可用的值如下
    + 获取年%Y，返回4位的整数
        + 获取年%y，返回2位的整数
        + 获取月%m，值为1-12的整数
    + 获取日%d，返回整数
        + 获取时%H，值为0-23的整数
        + 获取时%h，值为1-12的整数
        + 获取分%i，值为0-59的整数
        + 获取秒%s，值为0-59的整数
```
select date_format('2016-12-21','%Y %m %d');
```
+ 当前日期current_date()
```
select current_date();
```
+ 当前时间current_time()
```
select current_time();
```
+ 当前日期时间now()
```
select now();
```
### 视图
+ 对于复杂的查询，在多次使用后，维护是一件非常麻烦的事情
+ 解决：定义视图
+ 视图本质就是对查询的一个封装
+ 定义视图
```
create view stuscore as 
select students.*,scores.score from scores
inner join students on scores.stuid=students.id;
```
+ 视图的用途就是查询
```
select * from stuscore;
```
### 事务
+ 当一个业务逻辑需要多个sql完成时，如果其中某条sql语句出错，则希望整个操作都退回
+ 使用事务可以完成退回的功能，保证业务逻辑的正确性
+ 事务四大特性(简称ACID)
    + 原子性(Atomicity)：事务中的全部操作在数据库中是不可分割的，要么全部完成，要么均不执行
    + 一致性(Consistency)：几个并行执行的事务，其执行结果必须与按某一顺序串行执行的结果相一致
    + 隔离性(Isolation)：事务的执行不受其他事务的干扰，事务执行的中间结果对其他事务必须是透明的
    + 持久性(Durability)：对于任意已提交事务，系统必须保证该事务对数据库的改变不被丢失，即使数据库出现故障
+ 要求：表的类型必须是innodb或bdb类型，才可以对此表使用事务
+ 查看表的创建语句
```
show create table students;
```
+ 修改表的类型
```
alter table '表名' engine=innodb;
```
+ 事务语句
```
开启begin;
提交commit;
回滚rollback;
```
#### 示例1
+ 步骤1：打开两个终端，连接mysql，使用同一个数据库，操作同一张表
```
终端1：
select * from students;
------------------------
终端2：
begin;
insert into students(sname) values('张飞');
```
+ 步骤2
```
终端1：
select * from students;
```
+ 步骤3
```
终端2：
commit;
------------------------
终端1：
select * from students;
```
#### 示例2
+ 步骤1：打开两个终端，连接mysql，使用同一个数据库，操作同一张表
```
终端1：
select * from students;
------------------------
终端2：
begin;
insert into students(sname) values('张飞');
```
+ 步骤2
```
终端1：
select * from students;
```
+ 步骤3
```
终端2：
rollback;
------------------------
终端1：
select * from students;
```
### 索引
```
首先:先假设有一张student表,表的数据有10W条数据,
我们想执行下面语句。
SELECT * FROM student WHERE atitle = 'python'
在没有建立索引的时候，mysql执行这条语句是从10w条数据中一行一行的找。
当我们建立了索引那么mysql就直接根据 索引的列 找，而不需要一行行找，
那我们根据什么列来建立索引呢，根据实际的sql语句建立索引。
例如：
SELECT * FROM student WHERE atitle='mysql' and gender=0 and score>90
可以根据where条件来建立索引，例如上面的sql语句，我们可以建立，atitle，gender，score这三个列的索引。
这里要注意，如果语句是这么写的
SELECT * FROM student WHERE atitle='mysql' and score>90 and gender=0
那么一旦索引 找到是一个范围的时候，索引就没有用了。不是具体的某一块了，之后的索引建了也白建。
也就是说gender索引建了也白建。
如果语句这么写
SELECT * FROM student WHERE atitle='mysql' or score>90 and gender=0
那么or 后面的索引基本是废了,or语句没办法优化。
缺点：虽然索引大大提高了查询速度，同时会降低更新表的速度，如果对表进行insert，update，delete 
因为更新表，mysql不仅要保存数据，还要保存一下索引文件。
建立大量的索引会占用磁盘空间的索引文件。
```
#### 选择列的数据类型
```
越小的数据类型通常更好，能用int的不要用varchar，因为字符比较更为复杂，
尽量避免NULL：应该在创建表时指定列为NOT NULL，除非你想储存NULL,
在MYSQL中含有空值的列很难进行查询优化，因为它们使得索引，索引的统计信息以及运算更加复杂，
你应该用0，一个特殊的值，或者一个空串代替空值。
```
#### 操作
```
索引分单列索引和组合索引：
单列索引：即一个索引只包含单个列，一个表可以有多个单列索引，
组合索引：即一个索引包含多个列
查看索引：show index from tablename;
创建索引：create index indexname ON mytable(field(length)); 
length是创建表时设置的类型长度，int型可以不写。一个查询的条件尽量建立在一个索引上，
如有多个字段建立索引，则
mytable(field(length),field(length),field(length))
删除索引：drop index [indexname] on mytable;
```
```
1.开启运行时间监测：
set profiling=1;
2.查询
select * from areas where atitle='广州市';
3.查看执行时间
show profiles;
4.创建索引
create index atitleindex on areas(atitle(20));
5.再次查询
select * from areas where atitle='广州市';
6.再次查看索引时间
show profiles;
```

## 与python交互
### 安装
+ 安装mysql模块
```
sudo apt-get install python-mysqldb
```
+ python3中没有mysqldb  安装pymysql代替
```
pip3 install pymysql
```
### Connection对象
+ 用于建立与数据库的连接
+ 创建对象：调用connect()方法
```
conn=connect(参数列表)
```
+ 参数host：连接的mysql主机，如果本机是'localhost'
+ 参数port：连接的mysql主机的端口，默认是3306
+ 参数db：数据库的名称
+ 参数user：连接的用户名
+ 参数password：连接的密码
+ 参数charset：通信采用的编码方式，默认是'gb2312'，要求与数据库创建时指定的编码一致，否则中文会乱码

#### 对象的方法
+ close()关闭连接
+ commit()事务，所以需要提交才会生效
+ rollback()事务，放弃之前的操作
+ cursor()返回Cursor对象，用于执行sql语句并获得结果

### Cursor对象
+ 执行sql语句
+ 创建对象：调用Connection对象的cursor()方法
```
cursor1=conn.cursor()
```
#### 对象的方法
+ close()关闭
+ execute(operation [, parameters ])执行语句，返回受影响的行数
+ fetchone()执行查询语句时，获取查询结果集的第一个行数据，返回一个元组
+ next()执行查询语句时，获取当前行的下一行
+ fetchall()执行查询时，获取结果集的所有行，一行构成一个元组，再将这些元组装入一个元组返回
+ scroll(value[,mode])将行指针移动到某个位置
    + mode表示移动的方式
    + mode的默认值为relative，表示基于当前行移动到value，value为正则向下移动，value为负则向上移动
    + mode的值为absolute，表示基于第一条数据的位置，第一条数据的位置为0

#### 对象的属性
+ rowcount只读属性，表示最近一次execute()执行后受影响的行数
+ connection获得当前连接对象

### 增删改
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import pymysql

try:
    conn = pymysql.connect(host="192.168.211.134", port=3306, db="test2", user="root", passwd="mysql", charset="utf8")
    cs = conn.cursor()

    # 增加数据
    # count = cs.execute("insert into booktest_bookinfo values(0,'三国','1999,1,1',0,0,0)")

    # 修改数据
    # count = cs.execute("update booktest_bookinfo set btitle='三国演义' where id=6")

    # 删除数据
    # count = cs.execute("delete from booktest_bookinfo where id=6")


    # sql语句参数化 可用于过滤某些东西。
    btitle = input("请输入书名:")
    params = [btitle]
    count = cs.execute("insert into booktest_bookinfo values(0,%s,'1999,1,1',0,0,0)", params)

    print(count)
    conn.commit()
    cs.close()
    conn.close()

except Exception as e:
    print("错误原因："+ str(e))
```
#### 其它语句
+ cursor对象的execute()方法，也可以用于执行create table等语句
+ 建议在开发之初，就创建好数据库表结构，不要在这里执行

### 查询
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import pymysql

try:
    conn = pymysql.connect(host="192.168.211.134", port=3306, db="test2", user="root", passwd="mysql", charset="utf8")
    cs = conn.cursor()

    cs.execute("select * from booktest_bookinfo")

    # result = cs.fetchone() # 查询一条数据
    result = cs.fetchall() # 查询多行数据 返回的是一个元祖

    print(result)
    cs.close()
    conn.close()

except Exception as e:
    print("错误原因："+ str(e))
```

### 封装
+ 观察前面的文件发现，除了sql语句及参数不同，其它语句都是一样的
+ 创建MysqlHelper.py文件，定义类
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import pymysql


class MysqlHelper(object):
    def __init__(self, host, port, db, user, passwd, charset='utf8'):
        self.host = host
        self.port = port
        self.db = db
        self.user = user
        self.passwd = passwd
        self.charset = charset

    def open(self):
        self.conn = pymysql.connect(
            host=self.host, port=self.port,
            db=self.db, user=self.user,
            passwd=self.passwd, charset=self.charset
        )
        self.cursor = self.conn.cursor()

    def close(self):
        self.cursor.close()
        self.conn.close()

    def cud(self, sql, params=[]):
        try:
            self.open()
            self.cursor.execute(sql, params)
            self.conn.commit()
            self.close()
            return 1
        except Exception as e:
            print(e)
        return 0

    def all(self, sql, params=[]):
        try:
            self.open()
            self.cursor.execute(sql, params)
            result = self.cursor.fetchall()
            self.close()

            return result
        except Exception as e:
            print(e)

    def one(self, sql, params=[]):
        try:
            self.open()
            self.cursor.execute(sql, params)
            result = self.cursor.fetchone()
            self.close()

            return result
        except Exception as e:
            print(e)
```
+ 添加
+ 创建testInsertWrap.py文件，使用封装好的帮助类完成插入操作
```
from MysqlHelper import *

sql='insert into students(sname,gender) values(%s,%s)'
sname=input("请输入用户名：")
gender=input("请输入性别，1为男，0为女")
params=[sname,bool(gender)]

mysqlHelper=MysqlHelper('localhost',3306,'test1','root','mysql')
count=mysqlHelper.cud(sql,params)
if count==1:
    print('ok')
else:
    print('error')
```
+ 查询一个
```
sql='select sname,gender from students order by id desc'

helper=MysqlHelper('localhost',3306,'test1','root','mysql')
one=helper.one(sql)
print(one)
```

### 用户登录
#### 创建用户表userinfos
+ 表结构如下
    + id
    + uname
    + upwd
    + isdelete
+ 注意：需要对密码进行加密
+ 如果使用md5加密，则密码包含32个字符
+ 如果使用sha1加密，则密码包含40个字符，推荐使用这种方式
```
create table userinfos(
id int primary key auto_increment,
uname varchar(20),
upwd char(40),
isdelete bit default 0
);
```
+ 插入如下数据，用户名为123,密码为123,这是sha1加密后的值
```
insert into userinfos values(0,'123','40bd001563085fc35165329ea1ff5c5ecbdbbeef',0);
```
```
#!/usr/bin/env python
# -*- coding:utf-8 -*-

from mysql import MysqlHelper
import hashlib


def encrypt(pwd):
    s1 = hashlib.sha1()
    s1.update(bytes(pwd, encoding="utf-8"))
    return s1.hexdigest()

def login(sqlhelper):
    name = input("请输入用户名：")
    pwd = input("请输入密码：")

    sql = "SELECT passwd FROM users WHERE name=%s"
    result = sqlhelper.all(sql, [name])

    if len(result) == 0:
        print('用户名错误')
    elif result[0][0] == encrypt(pwd):
        print("登陆成功")
    else:
        print("密码错误")

def register(sqlhelper):
    name = input("请输入用户名：")
    pwd = input("请输入密码：")

    pwd2 = encrypt(pwd)

    sql = "select * from users where name=%s"
    result = sqlhelper.all(sql, [name])

    if len(result) == 0:
        sql = "insert into users(name,passwd) values(%s,%s)"
        sqlhelper.cud(sql, [name, pwd2])
    else:
        print("用户名重复")

def main():
    sqlhelper = MysqlHelper('192.168.211.133', 3306, 'python3', 'root', 'mysql')

    num = input("1.注册    2.登陆")

    if num == "1":
        register(sqlhelper)
    elif num == "2":
        login(sqlhelper)

if __name__ == "__main__":
    main()

```