---
title: Mysql报错注入原理分析（count()、rand()、group by）
date: 2018-08-17 14:44:08
tags:
	- Mysql
	- 注入
	- 渗透测试
categories: 渗透测试
keywords:
description:
---
Mysql报错注入原理分析(count()、rand()、group by)<!-- more -->

## 0x00 疑问
一直在用mysql数据库报错注入方法，但为何会报错？
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql043.png)

## 0x01 位置问题？
```
select count(*),(floor(rand(0)*2))x from information_schema.tables group by x;
```
这是网上最常见的语句,目前位置看到的网上sql注入教程,floor都是直接放count(*)后面，为了排除干扰，我们直接对比了两个报错语句，如下图:
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql044.png)

由上面的图片，可以知道报错跟位置无关。

## 0x02 绝对报错还是相对报错？
是不是报错语句有了floor(rand(0)*2)以及其他几个条件就一定报错？其实并不是如此，我们先建建个表，新增一条记录看看，如下图：
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql045.png)

确认表中只有一条记录后，再执行报错语句看看，如下图：
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql046.png)

多次执行均未发现报错。

然后我们新增一条记录。

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql047.png)

然后再测试下报错语句
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql048.png)

多次执行并没有报错

OK 那我们再增加一条

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql049.png)

执行报错语句

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql050.png)
ok 成功报错

由此可证明floor(rand(0)*2)报错是有条件的，记录必须3条以上，而且在3条以上必定报错，到底为何？请继续往下看。

## 0x03 随机因子具有决定权么(rand()和rand(0))

为了更彻底的说明报错原因，直接把随机因子去掉，再来一遍看看，先看一条记录的时候，如下图:

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql051.png)

一条记录的话 无论执行多少次也不报错
然后增加一条记录。
两条记录的话 结果就变成不确定性了

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql052.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql053.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql054.png)

随机出现报错。
然后再插入一条
三条记录之后，也和2条记录一样进行随机报错。

由此可见报错和随机因子是有关联的，但有什么关联呢，为什么直接使用rand()，有两条记录的情况下就会报错，而且是有时候报错，有时候不报错，而rand(0)的时候在两条的时候不报错，在三条以上就绝对报错？我们继续往下看。

## 0x04 不确定性与确定性

前面说过，floor(rand(0)*2)报错的原理是恰恰是由于它的确定性，这到底是为什么呢？从0x03我们大致可以猜想到，因为floor(rand()*2)不加随机因子的时候是随机出错的，而在3条记录以上用floor(rand(0)*2)就一定报错，由此可猜想floor(rand()*2)是比较随机的，不具备确定性因素，而floor(rand(0)*2)具备某方面的确定性。

为了证明我们猜想，分别对floor(rand()*2)和floor(rand(0)*2)在多记录表中执行多次(记录选择10条以上)，在有12条记录表中执行结果如下图：

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql055.png)

连续3次查询，毫无规则，接下来看看select floor(rand(0)*2) from test;，如下图：

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql056.png)

可以看到floor(rand(0)*2)是有规律的，而且是固定的，这个就是上面提到的由于是确定性才导致的报错，那为何会报错呢，我们接着往下看。

## 0x05 count与group by的虚拟表

使用select count(*) from test group by x;这种语句的时候我们经常可以看到下面类似的结果：

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql057.png)

可以看出 test12的记录有5条

与count(*)的结果相符合，那么mysql在遇到select count(*) from test group by x;这语句的时候到底做了哪些操作呢，我们果断猜测mysql遇到该语句时会建立一个虚拟表(实际上就是会建立虚拟表)，那整个工作流程就会如下图所示：

1、先建立虚拟表，如下图(其中key是主键，不可重复):
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql058.png)

2、开始查询数据，取数据库数据，然后查看虚拟表存在不，不存在则插入新记录，存在则count(*)字段直接加1，如下图:
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql059.png)

由此看到 如果key存在的话就+1， 不存在的话就新建一个key。

那这个和报错有啥内在联系，我们直接往下来，其实到这里，结合前面的内容大家也能猜个一二了。

## 0x06 floor(rand(0)*2)报错
其实mysql官方有给过提示，就是查询的时候如果使用rand()的话，该值会被计算多次，那这个“被计算多次”到底是什么意思，就是在使用group by的时候，floor(rand(0)*2)会被执行一次，如果虚表不存在记录，插入虚表的时候会再被执行一次，我们来看下floor(rand(0)*2)报错的过程就知道了，从0x04可以看到在一次多记录的查询过程中floor(rand(0)*2)的值是定性的，为011011…(记住这个顺序很重要)，报错实际上就是floor(rand(0)*2)被计算多次导致的，具体看看select count(*) from test group by floor(rand(0)*2);的查询过程：

1、查询前默认会建立空虚拟表如下图:
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql058.png)

2、取第一条记录，执行floor(rand(0)*2)，发现结果为0(第一次计算),查询虚拟表，发现0的键值不存在，则floor(rand(0)*2)会被再计算一次，结果为1(第二次计算)，插入虚表，这时第一条记录查询完毕，如下图:
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql060.png)

3、查询第二条记录，再次计算floor(rand(0)*2)，发现结果为1(第三次计算)，查询虚表，发现1的键值存在，所以floor(rand(0)*2)不会被计算第二次，直接count(*)加1，第二条记录查询完毕，结果如下:
![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql061.png)

4、查询第三条记录，再次计算floor(rand(0)*2)，发现结果为0(第4次计算)，查询虚表，发现键值没有0，则数据库尝试插入一条新的数据，在插入数据时floor(rand(0)*2)被再次计算，作为虚表的主键，其值为1(第5次计算)，然而1这个主键已经存在于虚拟表中，而新计算的值也为1(主键键值必须唯一)，所以插入的时候就直接报错了。

5、整个查询过程floor(rand(0)*2)被计算了5次，查询原数据表3次，所以这就是为什么数据表中需要3条数据，使用该语句才会报错的原因。

## 0x07 floor(rand()*2)报错

由0x05我们可以同样推理出不加入随机因子的情况，由于没加入随机因子，所以floor(rand()*2)是不可测的，因此在两条数据的时候，只要出现下面情况，即可报错，如下图:

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql062.png)

最重要的是前面几条记录查询后不能让虚表存在0,1键值，如果存在了，那无论多少条记录，也都没办法报错，因为floor(rand()*2)不会再被计算做为虚表的键值，这也就是为什么不加随机因子有时候会报错，有时候不会报错的原因。如图：

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql063.png)

当前面记录让虚表长成这样子后，由于不管查询多少条记录，floor(rand()*2)的值在虚表中都能找到，所以不会被再次计算，只是简单的增加count(*)字段的数量，所以不会报错，比如floor(rand(1)*2)，如图：

![](http://p3ek8hcdl.bkt.clouddn.com/image/sql/sql064.png)

在前两条记录查询后，虚拟表已经存在0和1两个键值了，所以后面再怎么弄还是不会报错。

总之报错需要count(*)，rand()、group by，三者缺一不可。