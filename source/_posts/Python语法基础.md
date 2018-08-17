---
title: Python语法基础
date: 2018-07-01 09:17:50
tags:
    - Python
    - 编程
    - 基础
    - 语法
categories: Python
---
![](http://p3ek8hcdl.bkt.clouddn.com/image/python.jpg)<!-- more -->

## 认识python和基础知识

### 认识python
Python是一种面向对象的解释型计算机程序设计语言，由荷兰人Guido van Rossum于1989年发明，第一个公开发行版发行于1991年。
Python是纯粹的自由软件， 源代码和解释器CPython遵循 GPL(GNU General Public License)协议。Python语法简洁清晰，特色之一是强制用空白符(white space)作为语句缩进。
Python具有丰富和强大的库。它常被昵称为胶水语言，能够把用其他语言制作的各种模块（尤其是C/C++）很轻松地联结在一起。常见的一种应用情形是，使用Python快速生成程序的原型（有时甚至是程序的最终界面），然后对其中有特别要求的部分，用更合适的语言改写，比如3D游戏中的图形渲染模块，性能要求特别高，就可以用C/C++重写，而后封装为Python可以调用的扩展类库。需要注意的是在您使用扩展类库时可能需要考虑平台问题，某些可能不提供跨平台的实现。

### 第一个python程序
#### 编写python程序方法1
打开“超级终端” 
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc001.png)

输入python3 ，输入```python3```表示用的python这门编程语言的第3个版本，如果只输入python的话表示用的是python的第2个版本
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc002.png)

输入以下代码
```
print('hello world')
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc003.png)
#### 编写python程序方法2
打开编辑软件```sublime```
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc004.png)
把以下代码，编写如下代码
![](http://p3ek8hcdl.bkt.clouddn.com/pyjc005.png)
保存代码
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc006.png)
运行程序
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc007.png)
#### 另外一种运行python的程序的方法
在代码第一行写入执行时的python解释器路径，编辑完后需要对此python文件添加'x'权限
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc008.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc009.png)
### 注释
通过用自己熟悉的语言，在程序中对某些代码进行标注说明，这就是注释的作用，能够大大增强程序的可读性
#### 单行注释
以#开头，#右边的所有东西当做说明，而不是真正要执行的程序，起辅助说明作用
```
# 我是注释，可以在里写一些功能说明之类的哦
print('hello world')
```
#### 多行注释
```
'''我是多行注释，可以写很多很多行的功能说明
        这就是我牛X指出

        哈哈哈。。。
'''
```
#### python程序中，中文支持
如果直接在程序中用到了中文，比如
```
print('你好')
```
如果直接运行输出，程序会出错：
![](http://p3ek8hcdl.bkt.clouddn.com/image/20180701103102.jpg)
解决的办法为：在程序的开头写入如下代码，这就是中文注释
```
#coding=utf-8
```
修改之后的程序:
```
#coding=utf-8
print('你好')
```
运行结果:
```
你好
```
在python的语法规范中推荐使用的方式：
```
# -*- coding:utf-8 -*-
```
### 变量以及类型
#### 变量的定义
在程序中，有时我们需要对2个数据进行求和，那么该怎样做呢？

大家类比一下现实生活中，比如去超市买东西，往往咱们需要一个菜篮子，用来进行存储物品，等到所有的物品都购买完成后，在收银台进行结账即可

如果在程序中，需要把2个数据，或者多个数据进行求和的话，那么就需要把这些数据先存储起来，然后把它们累加起来即可

在Python中，存储一个数据，需要一个叫做 变量 的东西，如下示例
```
num1 = 100 #num1就是一个变量，就好比一个小菜篮子

num2 = 87  #num2也是一个变量

result = num1 + num2 #把num1和num2这两个"菜篮子"中的数据进行累加，然后放到 result变量中
```
说明:

+ 所谓变量，可以理解为菜篮子，如果需要存储多个数据，最简单的方式是有多个变量，当然了也可以使用一个
+ 程序就是用来处理数据的，而变量就是用来存储数据的

#### 变量的类型
为了更充分的利用内存空间以及更有效率的管理内存，变量是有不同的类型的，如下所示:
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc010.png)

+ 怎样知道一个变量的类型呢？
    + 在python中，只要定义了一个变量，而且它有数据，那么它的类型就已经确定了，不需要咱们开发者主动的去说明它的类型，系统会自动辨别
    + 可以使用type(变量的名字)，来查看变量的类型

### 标示符和关键字
#### 标示符
什么是标示符，开发人员在程序中自定义的一些符号和名称，标示符是自己定义的,如变量名 、函数名等
#### 标示符的规则
+ 标示符由字母、下划线和数字组成，且数字不能开头
+ python中的标识符是区分大小写的

#### 命名规则
+ 见名知意
```
起一个有意义的名字，尽量做到看一眼就知道是什么意思(提高代码可 读性) 比如: 名字 就定义为 name , 定义学生 用 student
```
+ 驼峰命名法
```
小驼峰式命名法（lower camel case）： 第一个单词以小写字母开始；第二个单词的首字母大写，例如：myName、aDog

大驼峰式命名法（upper camel case）： 每一个单字的首字母都采用大写字母，例如：FirstName、LastName

不过在程序员中还有一种命名法比较流行，就是用下划线“_”来连接所有的单词，比如send_buf
```
#### 关键字
+ 什么是关键字
```
python一些具有特殊功能的标示符，这就是所谓的关键字

关键字，是python已经使用的了，所以不允许开发者自己定义和关键字相同的名字的标示符
```
+ 查看关键字:
```
and     as      assert     break     class      continue    def     del
elif    else    except     exec      finally    for         from    global
if      in      import     is        lambda     not         or      pass
print   raise   return     try       while      with        yield
```
可以通过以下命令进行查看当前系统中python的关键字
![](http://p3ek8hcdl.bkt.clouddn.com/image/pyjc011.png)

### 输出
#### 普通的输出
python中变量的输出
```
# 打印提示
print('hello world')
print('给我的卡---印度语，你好的意思')
```
#### 格式化输出
##### 格式化操作的目的
比如有以下代码:
```
pirnt("我今年10岁")
pirnt("我今年11岁")
pirnt("我今年12岁")
...
```
+ 想一想:
```
在输出年龄的时候，用了多次"我今年xx岁"，能否简化一下程序呢？？？
```
+ 答:
```
字符串格式化
```
##### 什么是格式化
看如下代码:
```
age = 10
print("我今年%d岁"%age)

age += 1
print("我今年%d岁"%age)

age += 1
print("我今年%d岁"%age)

...
```
在程序中，看到了%这样的操作符，这就是Python中格式化输出。
```
age = 18
name = "xiaohua"
print("我的姓名是%s,年龄是%d"%(name,age))
```
运行：
```
我的姓名是xiaohua,年龄是18
```
##### 常用的格式符号
下面是完整的，它可以与％符号使用列表:
|格式符号|转换|
|:---:|:---:|
|%c |字符|
|%s |通过str() 字符串转换来格式化|
|%i |有符号十进制整数|
|%d |有符号十进制整数|
|%u |无符号十进制整数|
|%o |八进制整数|
|%x |十六进制整数（小写字母）|
|%X |十六进制整数（大写字母）|
|%e |索引符号（小写'e'）|
|%E |索引符号（大写“E”）|
|%f |浮点实数|
|%g |％f和％e 的简写|
|%G |％f和％E的简写|

#### 换行输出
在输出的时候，如果有\n那么，此时\n后的内容会在另外一行显示

```
print("1234567890-------") # 会在一行显示

print("1234567890\n-------") # 一行显示1234567890，另外一行显示-------
```
### 输入
#### python2版本中
##### raw_input()
在Python中，获取键盘输入的数据的方法是采用 raw_input 函数（至于什么是函数，咱们以后的章节中讲解），那么这个 raw_input 怎么用呢?
看如下示例:
```
password = raw_input("请输入密码:")
print '您刚刚输入的密码是:', password
```
运行结果:

![](http://p3ek8hcdl.bkt.clouddn.com/image/py001.gif)
注意:

+ raw_input()的小括号中放入的是，提示信息，用来在获取数据之前给用户的一个简单提示
+ raw_input()在从键盘获取了数据以后，会存放到等号左边的变量中
+ raw_input()会把用户输入的任何值都作为字符串来对待

##### input()
input()函数与raw_input()类似，但其接受的输入必须是表达式。
```
>>> a = input() 
123
>>> a
123
>>> type(a)
<type 'int'>
>>> a = input()
abc
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<string>", line 1, in <module>
NameError: name 'abc' is not defined
>>> a = input()
"abc"
>>> a
'abc'
>>> type(a)
<type 'str'>
>>> a = input()
1+3
>>> a
4
>>> a = input()
"abc"+"def"
>>> a
'abcdef'
>>> value = 100
>>> a = input()
value
>>> a
100
```
input()接受表达式输入，并把表达式的结果赋值给等号左边的变量

#### python3版本中
没有raw_input()函数，只有input()
并且 python3中的input与python2中的raw_input()功能一样

### 运算符
python支持以下几种运算符

+ 算术运算符

下面以a=10 ,b=20为例进行计算

|运算符|描述|实例|
|:---:|:---:|:---:|
|+| 加|  两个对象相加 a + b 输出结果 30|
|-| 减|  得到负数或是一个数减去另一个数 a - b 输出结果 -10|
|*| 乘|  两个数相乘或是返回一个被重复若干次的字符串 a * b 输出结果 200|
|/| 除|  x除以y b / a 输出结果 2|
|//|    取整除|    返回商的整数部分 9//2 输出结果 4 , 9.0//2.0 输出结果 4.0|
|%| 取余| 返回除法的余数 b % a 输出结果 0|
|**|    幂|  返回x的y次幂 a**b 为10的20次方， 输出结果 100000000000000000000|

```
>>> 9/2.0
4.5
>>> 9//2.0
4.0
```
+ 赋值运算符

|运算符|描述|实例|
|:---:|:---:|:---:|
|=| 赋值运算符|  把=号右边的结果给左边的变量 num=1+2*3 结果num的值为7|

```
>>> a, b = 1, 2
>>> a
1
>>> b
2
```
+ 复合赋值运算符

|运算符|描述|实例|
|:---:|:---:|:---:|
|+=|    加法赋值运算符|    c += a 等效于 c = c + a|
|-=|    减法赋值运算符|    c -= a 等效于 c = c - a|
|*=|    乘法赋值运算符|    c *= a 等效于 c = c * a|
|/=|    除法赋值运算符|    c /= a 等效于 c = c / a|
|%=|    取模赋值运算符|    c %= a 等效于 c = c % a|
|**=|   幂赋值运算符| c **= a 等效于 c = c ** a|
|//=|   取整除赋值运算符|   c //= a 等效于 c = c // a|

### 数据类型转换
#### 常用的数据类型转换
|函数|说明|
|:---:|:---:|
|int(x [,base ])|   将x转换为一个整数|
|long(x [,base ])|  将x转换为一个长整数|
|float(x )| 将x转换到一个浮点数|
|complex(real [,imag ])|    创建一个复数|
|str(x )|   将对象 x 转换为字符串|
|repr(x )|  将对象 x 转换为表达式字符串|
|eval(str )|    用来计算在字符串中的有效Python表达式,并返回一个对象|
|tuple(s )| 将序列 s 转换为一个元组|
|list(s )|  将序列 s 转换为一个列表|
|chr(x )|   将一个整数转换为一个字符|
|unichr(x )|    将一个整数转换为Unicode字符|
|ord(x )|   将一个字符转换为它的整数值|
|hex(x )|   将一个整数转换为一个十六进制字符串|
|oct(x )|   将一个整数转换为一个八进制字符串|

举例
```
a = '100' # 此时a的类型是一个字符串，里面存放了100这3个字符
b = int(a) # 此时b的类型是整型，里面存放的是数字100

print("a=%d"%b)
```
### 判断语句介绍
#### 开发中的判断场景
密码判断
```
if 输入的密码 等于 正确密码:
    登陆成功
else:
    登陆失败
```
重要日期判断
```
if 今天是周六或者周日:
        约妹子

    if 今天是情人节:
        买玫瑰

    if 今天发工资:

        先还信用卡的钱

        if 有剩余:

            又可以happy了，O(∩_∩)O哈哈~

        else:

            噢，no。。。还的等30天
```

小总结：

+ 如果某些条件满足，才能做某件事情，而不满足时不允许做，这就是所谓的判断
+ 不仅生活中有，在软件开发中“判断”功能也经常会用到

### if判断语句
#### if判断语句介绍
+ if语句是用来进行判断的，其使用格式如下：
```
if 要判断的条件:
    条件成立时，要做的事情
```
+ demo1:
```
age = 30

print "------if判断开始------"

if age>=18:
    print "我已经成年了"

print "------if判断结束------"
```
+ 运行结果:
```
------if判断开始------
我已经成年了
------if判断结束------
```
+ demo2:
```
age = 16

print "------if判断开始------"

if age>=18:
    print "我已经成年了"

print "------if判断结束------"
```
+ 运行结果:
```
------if判断开始------
------if判断结束------
```
小总结：

+ 以上2个demo仅仅是age变量的值不一样，结果却不同；能够看得出if判断语句的作用：就是当满足一定条件时才会执行那块代码，否则就不执行那块代码

注意：

+ 代码的缩进为一个tab键，或者4个空格

#### 练一练
要求：从键盘获取自己的年龄，判断是否大于或者等于18岁，如果满足就输出“哥，已成年，网吧可以去了”

+ 使用input从键盘中获取数据，并且存入到一个变量中
+ 使用if语句，来判断 age>=18是否成立

```
age = input("请输入年龄：")
if int(age) >= 18:
    print("哥，已成年，网吧可以去了")
```
### 比较、关系运算符
#### 比较(即关系)运算符
python中的比较运算符如下表

|运算符|描述|示例|
|:---:|:---:|:---:|
|== |检查两个操作数的值是否相等，如果是则条件变为真。|  如a=3,b=3则（a == b) 为 true.|
|!= |检查两个操作数的值是否相等，如果值不相等，则条件变为真。   |如a=1,b=3则(a != b) 为 true.|
|<> |检查两个操作数的值是否相等，如果值不相等，则条件变为真。   |如a=1,b=3则(a <> b) 为 true。这个类似于 != 运算符|
|>  |检查左操作数的值是否大于右操作数的值，如果是，则条件成立。  |如a=7,b=3则(a > b) 为 true.|
|<  |检查左操作数的值是否小于右操作数的值，如果是，则条件成立。  |如a=7,b=3则(a < b) 为 false.|
|>= |检查左操作数的值是否大于或等于右操作数的值，如果是，则条件成立。   |如a=3,b=3则(a >= b) 为 true.|
|<= |检查左操作数的值是否小于或等于右操作数的值，如果是，则条件成立。   |如a=3,b=3则(a <= b) 为 true.|

#### 逻辑运算符
a=10 b=20

|运算符|逻辑表达式|描述|示例|
|:---:|:---:|:---:|:---:|
|and|   x and y |布尔"与" - 如果 x 为 False，x and y 返回 False，否则它返回 y 的计算值。|   (a and b) 返回 20。|
|or |x or y |布尔"或" - 如果 x 是 True，它返回 True，否则它返回 y 的计算值。|    (a or b) 返回 10。|
|not|   not x|  布尔"非" - 如果 x 为 True，返回 False 。如果 x 为 False，它返回 True。|   not(a and b) 返回 False|

## 判断语句和循环语句
### if-else

>想一想：在使用if的时候，它只能做到满足条件时要做的事情。那万一需要在不满足条件的时候，做某些事，该怎么办呢？

>答：else

#### if-else的使用格式
```
if 条件:
    满足条件时要做的事情1
    满足条件时要做的事情2
    满足条件时要做的事情3
    ...(省略)...
else:
    不满足条件时要做的事情1
    不满足条件时要做的事情2
    不满足条件时要做的事情3
    ...(省略)...
```
demo1
```
chePiao = 1 # 用1代表有车票，0代表没有车票
if chePiao == 1:
    print("有车票，可以上火车")
    print("终于可以见到Ta了，美滋滋~~~")
else:
    print("没有车票，不能上车")
    print("亲爱的，那就下次见了，一票难求啊~~~~(>_<)~~~~")
```
结果1：有车票的情况
```
有车票，可以上火车
终于可以见到Ta了，美滋滋~~~
```
结果2：没有车票的情况
```
没有车票，不能上课
亲爱的，那就下次见了，一票难求啊~~~~(>_<)~~~~
```

### elif
想一想:
>if能完成当xxx时做事情

>if-else能完成当xxx时做事情1，否则做事情2

>如果有这样一种情况：当xxx1时做事情1，当xxx2时做事情2，当xxx3时做事情3，那该怎么实现呢？

答:
>elif

#### elif的功能
elif的使用格式如下:
```
if xxx1:
    事情1
elif xxx2:
    事情2
elif xxx3:
    事情3
```
说明:

+ 当xxx1满足时，执行事情1，然后整个if结束
+ 当xxx1不满足时，那么判断xxx2，如果xxx2满足，则执行事情2，然后整个if结束
+ 当xxx1不满足时，xxx2也不满足，如果xxx3满足，则执行事情3，然后整个if结束

demo:
```
score = 77

if score>=90 and score<=100:
    print('本次考试，等级为A')
elif score>=80 and score<90:
    print('本次考试，等级为B')
elif score>=70 and score<80:
    print('本次考试，等级为C')
elif score>=60 and score<70:
    print('本次考试，等级为D')
elif score>=0 and score<60:
    print('本次考试，等级为E')
```
#### 注意点
可以和else一起使用
```
if 性别为男性:
    输出男性的特征
    ...
elif 性别为女性:
    输出女性的特征
    ...
else:
    第三种性别的特征
    ...
```

说明:

+ 当 “性别为男性” 满足时，执行 “输出男性的特征”的相关代码
+ 当 “性别为男性” 不满足时，如果 “性别为女性”满足，则执行 “输出女性的特征”的相关代码
+ 当 “性别为男性” 不满足，“性别为女性”也不满足，那么久默认执行else后面的代码，即 “第三种性别的特征”相关代码

elif必须和if一起使用，否则出错

### if嵌套
通过学习if的基本用法，已经知道了

+ 当需要满足条件去做事情的这种情况需要使用if
+ 当满足条件时做事情A，不满足条件做事情B的这种情况使用if-else

想一想：
>坐火车或者地铁的实际情况是：先进行安检如果安检通过才会判断是否有车票，或者是先检查是否有车票之后才会进行安检，即实际的情况某个判断是再另外一个判断成立的基础上进行的，这样的情况该怎样解决呢？
答：
>if嵌套
#### if嵌套的格式
```
if 条件1:
    满足条件1 做的事情1
    满足条件1 做的事情2
    ...(省略)...

    if 条件2:
        满足条件2 做的事情1
        满足条件2 做的事情2
        ...(省略)...
```
说明

+ 外层的if判断，也可以是if-else
+ 内层的if判断，也可以是if-else
+ 根据实际开发的情况，进行选择

#### if嵌套的应用
demo：

```
chePiao = 1     # 用1代表有车票，0代表没有车票
daoLenght = 9     # 刀子的长度，单位为cm

if chePiao == 1:
    print("有车票，可以进站")
    if daoLenght < 10:
        print("通过安检")
        print("终于可以见到Ta了，美滋滋~~~")
    else:
        print("没有通过安检")
        print("刀子的长度超过规定，等待警察处理...")
else:
    print("没有车票，不能进站")
    print("亲爱的，那就下次见了，一票难求啊~~~~(>_<)~~~~")
```

结果1：chePiao = 1;daoLenght = 9

```
有车票，可以进站
通过安检
终于可以见到Ta了，美滋滋~~~
```

结果2：chePiao = 1;daoLenght = 20

```
有车票，可以进站
没有通过安检
刀子的长度超过规定，等待警察处理...
```

结果3：chePiao = 0;daoLenght = 9

```
没有车票，不能进站
亲爱的，那就下次见了，一票难求啊~~~~(>_<)~~~~
```

结果4：chePiao = 0;daoLenght = 20

```
没有车票，不能进站
亲爱的，那就下次见了，一票难求啊~~~~(>_<)~~~~
```

想一想:为什么结果3和结果4相同？？？

### if应用:猜拳游戏
#### 运行效果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/jdfwpy.gif)

#### 参考代码:
```
import random

player = input('请输入：剪刀(0)  石头(1)  布(2):')

player = int(player)

computer = random.randint(0,2)

# 用来进行测试
#print('player=%d,computer=%d',(player,computer))

if ((player == 0) and (computer == 2)) or ((player ==1) and (computer == 0)) or ((player == 2) and (computer == 1)):
    print('获胜，哈哈，你太厉害了')
elif player == computer:
    print('平局，要不再来一局')
else:
    print('输了，不要走，洗洗手接着来，决战到天亮')
```

### 循环语句介绍
#### 软件开发中循环的使用场景
跟媳妇承认错误，说一万遍"媳妇儿，我错了"
```
print("媳妇儿，我错了")
print("媳妇儿，我错了")
print("媳妇儿，我错了")
...(还有99997遍)...
```
使用循环语句一句话搞定
```
i = 0
while i<10000:
    print("媳妇儿，我错了")
    i+=1
```
#### 小总结

+ 一般情况下，需要多次重复执行的代码，都可以用循环的方式来完成
+ 循环不是必须要使用的，但是为了提高代码的重复使用率，所以有经验的开发者都会采用循环

### while循环
#### while循环的格式
```
while 条件:
    条件满足时，做的事情1
    条件满足时，做的事情2
    条件满足时，做的事情3
    ...(省略)...
```
demo
```
i = 0
while i<5:
    print("当前是第%d次执行循环"%(i+1))
    print("i=%d"%i)
    i+=1
```
结果:
```
当前是第1次执行循环
i=0
当前是第2次执行循环
i=1
当前是第3次执行循环
i=2
当前是第4次执行循环
i=3
当前是第5次执行循环
i=4
```
### while循环应用
#### 计算1~100的累积和（包含1和100）
参考代码如下:
```
#encoding=utf-8

i = 1
sum = 0
while i<=100:
    sum = sum + i
    i += 1

print("1~100的累积和为:%d"%sum)
```
#### 计算1~100之间偶数的累积和（包含100）
参考代码如下:
```
#encoding=utf-8

i = 1
sum = 0
while i<=100:
    if i%2 == 0:
        sum = sum + i
    i+=1

print("1~100的累积和为:%d"%sum)
```

### while循环嵌套

+ 前面学习过if的嵌套了，想一想if嵌套是什么样子的？
+ 类似if的嵌套，while嵌套就是：while里面还有while

#### while嵌套的格式
```
while 条件1:

    条件1满足时，做的事情1
    条件1满足时，做的事情2
    条件1满足时，做的事情3
    ...(省略)...

    while 条件2:
        条件2满足时，做的事情1
        条件2满足时，做的事情2
        条件2满足时，做的事情3
        ...(省略)...
```
#### while嵌套应用一
要求：打印如下图形：
```
*
* *
* * *
* * * *
* * * * *
```
参考代码：
```
i = 1
while i<=5:

    j = 1
    while j<=i:
        print("* ",end='')
        j+=1

    print("\n")
    i+=1
```
#### while嵌套应用二：九九乘法表

![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20161017_87.png)

参考代码：
```
i = 1
while i<=9:
    j=1
    while j<=i:
        print("%d*%d=%-2d "%(j,i,i*j),end='')
        j+=1
    print('\n')
    i+=1
```
### for循环
像while循环一样，for可以完成循环的功能。
在Python中 for循环可以遍历任何序列的项目，如一个列表或者一个字符串等。

#### for循环的格式
```
for 临时变量 in 列表或者字符串等:
    循环满足条件时执行的代码
else:
    循环不满足条件时执行的代码
```
#### demo1
```
name = 'ixysec'

for x in name:
    print(x)
```
运行结果如下:
![](http://p3ek8hcdl.bkt.clouddn.com/image/20180702211608.jpg)

#### demo2
```
name = ''

for x in name:
    print(x)
else:
    print("没有数据")
```
运行结果如下:
![](http://p3ek8hcdl.bkt.clouddn.com/image/20180702211958.jpg)

### break和continue
#### break
##### for循环
普通的循环示例如下：
```
name = 'ixysec'
for x in name:
    print('----')
    print(x)
```
运行结果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/20180702212315.jpg)

带有break的循环示例如下:
```
name = 'ixysec'

for x in name:
    print('----')
    if x == 'y': 
        break
    print(x)
```
运行结果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/20180702212615.jpg)

##### while循环
普通的循环示例如下：
```
i = 0

while i<10:
    i = i+1
    print('----')
    print(i)
```
运行结果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20161017_94.png)

带有break的循环示例如下:
```
i = 0

while i<10:
    i = i+1
    print('----')
    if i==5:
        break
    print(i)
```
运行结果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20161017_93.png)
##### 小总结:
break的作用：用来结束整个循环
#### continue
##### for循环
带有continue的循环示例如下:
```
name = 'dongGe'

for x in name:
    print('----')
    if x == 'g': 
        continue
    print(x)
```
运行结果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20161017_95.png)
##### while循环
带有continue的循环示例如下:
```
i = 0

while i<10:
    i = i+1
    print('----')
    if i==5:
        continue
    print(i)
```
运行结果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20161017_96.png)
##### 小总结:
continue的作用：用来结束本次循环，紧接着执行下一次的循环

#### 注意点

+ break/continue只能用在循环中，除此以外不能单独使用
+ break/continue在嵌套循环中，只对最近的一层循环起作用

## 字符串、列表、元组、字典
### 字符串介绍
想一想：
>当打来浏览器登录某些网站的时候，需要输入密码，浏览器把密码传送到服务器后，服务器会对密码进行验证，其验证过程是把之前保存的密码与本次传递过去的密码进行对比，如果相等，那么就认为密码正确，否则就认为不对；服务器既然想要存储这些密码可以用数据库（比如MySQL），当然为了简单起见，咱们可以先找个变量把密码存储起来即可；那么怎样存储带有字母的密码呢？

答：
>字符串

#### python中字符串的格式
如下定义的变量a，存储的是数字类型的值
```
a = 100
```
如下定义的变量b，存储的是字符串类型的值
```
b = "hello ixysec"
或者
b = 'hello ixysec'
```
小总结：

+ 双引号或者单引号中的数据，就是字符串

### 字符串输出
demo 
```
name = 'laot'
position = '学生'
address = '辽宁鞍山'

print('--------------------------------------------------')
print("姓名：%s"%name)
print("职位：%s"%position)
print("地址：%s"%address)
print('--------------------------------------------------')
```
结果:
```
--------------------------------------------------
姓名：laot
职位：学生
地址：辽宁鞍山
-------------------------------------------------
```
### 字符串输入
之前在学习input的时候，通过它能够完成从键盘获取数据，然后保存到指定的变量中；
注意：input获取的数据，都以字符串的方式进行保存，即使输入的是数字，那么也是以字符串方式保存

demo:
```
userName = input('请输入用户名:')
print("用户名为：%s"%userName)

password = input('请输入密码:')
print("密码为：%s"%password)
```
结果：（根据输入的不同结果也不同）
![](http://p3ek8hcdl.bkt.clouddn.com/image/jdfwstr.gif)

### 下标和切片
#### 下标索引
所谓“下标”，就是编号，就好比超市中的存储柜的编号，通过这个编号就能找到相应的存储空间

字符串中"下标"的使用
列表与元组支持下标索引好理解，字符串实际上就是字符的数组，所以也支持下标索引。
如果有字符串:name = 'abcdef'，在内存中的实际存储如下:

![](http://p3ek8hcdl.bkt.clouddn.com/image/strxb001.png)

如果想取出部分字符，那么可以通过`下标`的方法，（注意python中下标从 0 开始）

```
name = 'abcdef'

print(name[0])
print(name[1])
print(name[2])
```
运行结果: 
![](http://p3ek8hcdl.bkt.clouddn.com/image/strxb002.png)

#### 切片
切片是指对操作的对象截取其中一部分的操作。字符串、列表、元组都支持切片操作。

+ 切片的语法：[起始:结束:步长]
+ 注意：选取的区间属于左闭右开型，即从"起始"位开始，到"结束"位的前一位结束（不包含结束位本身)。

我们以字符串为例讲解。
如果取出一部分，则可以在中括号[]中，使用:

```
name = 'abcdef'

print(name[0:3]) # 取 下标0~2 的字符
```
运行结果:
![](http://p3ek8hcdl.bkt.clouddn.com/image/strxb003.png)

```
name = 'abcdef'

print(name[3:5]) # 取 下标为3、4 的字符
```
运行结果：
![](http://p3ek8hcdl.bkt.clouddn.com/image/strxb004.png)

```
name = 'abcdef'

print(name[2:]) # 取 下标为2开始到最后的字符
```
运行结果：
![](http://p3ek8hcdl.bkt.clouddn.com/image/strxb005.png)
```
 >>> a = "abcdef"
 >>> a[:3]
 'abc'
 >>> a[::2]
 'ace'
 >>> a[5:1:2] 
 ''
 >>> a[1:5:2]
 'bd'
 >>> a[::-2]
 'fdb' 
 >>> a[5:1:-2]
 'fd'
```

### 字符串常见操作
如有字符串mystr = 'hello world itcast and itcastcpp'，以下是常见的操作

#### <1>find
检测 str 是否包含在 mystr中，如果是返回开始的索引值，否则返回-1
```
mystr.find(str, start=0, end=len(mystr))
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_215.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_216.png)

#### <2>index
跟find()方法一样，只不过如果str不在 mystr中会报一个异常.
```
mystr.index(str, start=0, end=len(mystr))
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_217.png)

#### <3>count
返回 str在start和end之间 在 mystr里面出现的次数
```
mystr.count(str, start=0, end=len(mystr))
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_218.png)

#### <4>replace
把 mystr 中的 str1 替换成 str2,如果 count 指定，则替换不超过 count 次.
```
mystr.replace(str1, str2,  mystr.count(str1))
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_214.png)

#### <5>split
以 str 为分隔符切片 mystr，如果 maxsplit有指定值，则仅分隔 maxsplit 个子字符串
```
mystr.split(str=" ", 2)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_211.png)

#### <6>capitalize
把字符串的第一个字符大写
```
mystr.capitalize()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_2191.png)

#### <7>title
把字符串的每个单词首字母大写
```
>>> a = "hello ixysec"
>>> a.title()
'Hello Ixysec'
```

#### <8>startswith
检查字符串是否是以 obj 开头, 是则返回 True，否则返回 False
```
mystr.startswith(obj)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_221.png)

#### <9>endswith
检查字符串是否以obj结束，如果是返回True,否则返回 False.
```
mystr.endswith(obj)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_222.png)

#### <10>lower
转换 mystr 中所有大写字符为小写
```
mystr.lower()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_223.png)

#### <11>upper
转换 mystr 中的小写字母为大写
```
mystr.upper()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20aaaa4.png)

#### <12>ljust
返回一个原字符串左对齐,并使用空格填充至长度 width 的新字符串
```
mystr.ljust(width) 
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_225.png)

#### <13>rjust
返回一个原字符串右对齐,并使用空格填充至长度 width 的新字符串
```
mystr.rjust(width)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_226.png)

#### <14>center
返回一个原字符串居中,并使用空格填充至长度 width 的新字符串
```
mystr.center(width)
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_220.png)

#### <15>lstrip
删除 mystr 左边的空白字符
```
mystr.lstrip()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_227.png)

#### <16>rstrip
删除 mystr 字符串末尾的空白字符
```
mystr.rstrip()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/Snip20160814_228.png)

#### <17>strip
删除mystr字符串两端的空白字符
```
>>> a = "\n\t itcast \t\n"
>>> a.strip()
'itcast'
```

#### <18>rfind
类似于 find()函数，不过是从右边开始查找.
```
mystr.rfind(str, start=0,end=len(mystr))
```

#### <19>rindex
类似于 index()，不过是从右边开始.
```
mystr.rindex( str, start=0,end=len(mystr))
```

#### <20>partition
把mystr以str分割成三部分,str前，str和str后
```
mystr.partition(str)
```

#### <21>rpartition
类似于 partition()函数,不过是从右边开始.
```
mystr.rpartition(str)
```

#### <22>splitlines
按照行分隔，返回一个包含各行作为元素的列表
```
mystr.splitlines()
```

#### <23>isalpha
如果 mystr 所有字符都是字母 则返回 True,否则返回 False
```
mystr.isalpha()
```

#### <24>isdigit
如果 mystr 只包含数字则返回 True 否则返回 False.
```
mystr.isdigit()
```

#### <25>isalnum
如果 mystr 所有字符都是字母或数字则返回 True,否则返回 False
```
mystr.isalnum()
```

#### <26>isspace
如果 mystr 中只包含空格，则返回 True，否则返回 False
```
mystr.isspace()
```

#### <27>join
mystr 中每个字符后面插入str,构造出一个新的字符串
```
mystr.join(str)
```

### 列表介绍





