---
title: Django
date: 2018-01-30 14:41:56
tags:
	- python
	- django
categories: Python
---
![](http://p3ek8hcdl.bkt.clouddn.com/image/timgsss.jpg)<!-- more -->
## 简介
***
+ MVC
``` bash
大部分开发语言中都有MVC框架
MVC框架的核心思想是：解耦
降低各功能模块之间的耦合性，方便变更，更容易重构代码，最大程度上实现代码的重用

m表示model，主要用于对数据库层的封装
v表示view，用于向用户展示结果
c表示controller，是核心，用于处理请求、获取数据、返回结果
```
+ MVT
```bash
Django是一款python的web开发框架
与MVC有所不同，属于MVT框架 
m表示model，负责与数据库交互 
v表示view，是核心，负责接收请求、获取数据、返回结果 
t表示template，负责呈现内容到浏览器
```

## 入门
### 环境搭建
ubuntu16.04

#### python虚拟环境搭建
```
安装虚拟环境virtualenv
sudo pip install virtualenv

安装虚拟环境管理
sudo easy_install virtualenvwrapper

创建目录用来存放虚拟环境
mkdir $HOME/.virtualenvs

在~/.bashrc中添加行：
    export WORKON_HOME=$HOME/.virtualenvs
    source /usr/local/bin/virtualenvwrapper.sh
    
运行:
    source ~/.bashrc
```
```
mkvirtualenv [虚拟环境名称]  创建python虚拟环境
mkvirtualenv -p /usr/bin/python3(python安装目录，这里写的是ubuntu默认安装路径，) [虚拟环境名称]
mkvirtualenv -p /usr/bin/python2 [虚拟环境名称]  
-p 可以指定python2解释器还是python3解释器。
workon [虚拟环境名称]   选择python虚拟环境
deactivate         退出虚拟环境 
rmvirtualenv [虚拟环境名称]   删除(慎用)
```
查看虚拟环境中已经安装的包
```
pip list
pip freeze
```
#### 安装django
```
安装1.8.2版本，稳定性高、使用广、文档多的版本
pip install django==1.8.2
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/django.jpg)
#### 创建项目
```
django-admin startproject test1
```
进入test1目录，目录结构如下图：
![](http://p3ek8hcdl.bkt.clouddn.com/image/test1.png)
#### 目录说明
```
manage.py：一个命令行工具，可以使你用多种方式对Django项目进行交互
_init _.py：一个空文件，它告诉Python这个目录应该被看做一个Python包
settings.py：项目的配置
urls.py：项目的URL声明
wsgi.py：项目与WSGI兼容的Web服务器入口
```
### 设计模型
```
本示例完成“图书-英雄”信息的维护，需要存储两种数据：图书、英雄
```
```
图书表结构设计：
表名：BookInfo
图书名称：btitle
图书发布时间：bpub_date
```
```
英雄表结构设计：
表名：HeroInfo
英雄姓名：hname
英雄性别：hgender
英雄简介：hcontent
所属图书：hbook

图书-英雄的关系为一对多
例： 天龙八部——虚竹
     天龙八部——乔峰
     天龙八部——王姑娘
```
#### 数据库配置
```
在settings.py文件中，通过DATABASES项进行数据库设置
django支持的数据库包括：sqlite、mysql等主流数据库
Django默认使用SQLite数据库
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/djangodbs.jpg)
```
setting.py 
mysql  配置
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME' : 'test1' ,
        'USER' : 'root' ,
        'PASSWORD' : 'mysql' ,
        'HOST' : '127.0.0.1' ,
        'PORT' : '3306' ,
    }
}
```
python3中已经移除mysqldb，需先安装pymysql
```
pip install pymysql
找到项目目录下的__init__.py 加入如下代码。
import pymysql  
pymysql.install_as_MySQLdb()
```
#### 创建应用
```
在一个项目中可以创建一到多个应用，每个应用进行一种业务处理
创建应用的命令：
python manage.py startapp booktest

├── booktest
│   ├── admin.py
│   ├── __init__.py
│   ├── migrations
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
└── test1
    ├── __init__.py
    ├── __init__.pyc
    ├── settings.py
    ├── settings.pyc
    ├── urls.py
    └── wsgi.py
```
#### 定义模型类
+ 有一个数据表，就有一个模型类与之对应
+ 打开models.py文件，定义模型类
+ 引入包from django.db import models
+ 模型类继承自models.Model类
+ 说明：不需要定义主键列，在生成时会自动添加，并且值为自动增长
+ 当输出对象时，会调用对象的str方法
``` python
from django.db import models


class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateTimeField()

    def __str__(self):
        return "%d" % self.pk


class HeroInfo(models.Model):
    hname = models.CharField(max_length=20)
    hgender = models.BooleanField()
    hcontent = models.CharField(max_length=100)
    hbook = models.ForeignKey('BookInfo')

    def __str__(self):
        return "%d" % self.pk
```
#### 生成数据表
```
激活模型：编辑settings.py文件，将booktest应用加入到installed_apps中
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'booktest',
)
```
```
生成迁移文件：根据模型类生成sql语句
python manage.py makemigrations
迁移文件被生成到应用的migrations目录
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/migrate.png)
```
执行迁移：执行sql语句生成数据表
python manage.py migrate

mysql> show tables;
+----------------------------+
| Tables_in_test1            |
+----------------------------+
| auth_group                 |
| auth_group_permissions     |
| auth_permission            |
| auth_user                  |
| auth_user_groups           |
| auth_user_user_permissions |
| booktest_bookinfo          |
| booktest_heroinfo          |
| django_admin_log           |
| django_content_type        |
| django_migrations          |
| django_session             |
+----------------------------+

```
#### 测试数据操作
```
进入python shell，进行简单的模型API练习
python manage.py shell
```

引入需要的包：
```
from booktest.models import BookInfo,HeroInfo
from django.utils import timezone
from datetime import *
```
查询所有图书信息：
```
BookInfo.objects.all()
```
新建图书信息：
```
b = BookInfo()
b.btitle="射雕英雄传"
b.bpub_date=datetime(year=1990,month=1,day=10)
b.save()
```
查找图书信息：
```
b=BookInfo.objects.get(pk=1) # 获取主键为1的数据
```
输出图书信息：
```
b
b.id
b.btitle
```
修改图书信息：
```
b.btitle="天龙八部"
b.save()
```
删除图书信息：
```
b.delete()
```
#### 关联对象的操作
```
对于HeroInfo可以按照上面的操作方式进行
添加，注意添加关联对象

h=HeroInfo()
h.hname='虚竹'
h.hgender=True
h.hcontent='天山六阳掌'
h.hbook=b
h.save()

获得关联集合：返回当前book对象的所有hero
b.heroinfo_set.all()

有一个HeroInfo存在，必须要有一个BookInfo对象，提供了创建关联的数据：
h=b.heroinfo_set.create(hname='乔峰',hgender=False,hcontent='降龙十八掌')
```
### 管理站点
#### 服务器
```
运行如下命令可以开启服务器
python manage.py runserver ip:port
可以不写ip，默认端口为8000

可以修改端口
python manage.py runserver 8080

这是一个纯python编写的轻量级web服务器，仅在开发阶段使用
如果修改文件不需要重启服务器，如果增删文件需要重启服务器
```
#### 管理操作
```
站点分为“内容发布”和“公共访问”两部分
“内容发布”的部分负责添加、修改、删除内容，开发这些重复的功能是一件单调乏味、缺乏创造力的工作。
为此，Django会根据定义的模型类完全自动地生成管理模块
```
+ 使用django的管理
```
创建一个管理员用户
python manage.py createsuperuser
按提示输入用户名、邮箱、密码

启动服务器，通过“127.0.0.1:8000/admin”访问，输入上面创建的用户名、密码完成登录
进入管理站点，默认可以对groups、users进行管理
```
+ 管理界面本地化
```
编辑settings.py文件，设置编码、时区
LANGUAGE_CODE = 'zh-Hans'
TIME_ZONE = 'Asia/Shanghai'
```

+ 向admin注册booktest的模型
```
打开booktest/admin.py文件，注册模型
from django.contrib import admin
from models import BookInfo
admin.site.register(BookInfo)

刷新管理页面，可以对BookInfo的数据进行增删改查操作
问题：如果在str方法中返回中文，在修改和添加时会报ascii的错误
解决：在str()方法中，将字符串末尾添加“.encode('utf-8')”
```
+ 自定义管理页面
```
Django提供了admin.ModelAdmin类
通过定义ModelAdmin的子类，来定义模型在Admin界面的显示方式
class BookInfoAdmin(admin.ModelAdmin):
    ...
admin.site.register(Question, BookInfoAdmin)
```
列表页属性
```
list_display：显示字段，可以点击列头进行排序
list_display = ['pk', 'btitle', 'bpub_date']

list_filter：过滤字段，过滤框会出现在右侧
list_filter = ['btitle']

search_fields：搜索字段，搜索框会出现在上侧
search_fields = ['btitle']

list_per_page：分页，分页框会出现在下侧
list_per_page = 10
```
添加、修改页属性
```
fields：属性的先后顺序
fields = ['bpub_date', 'btitle']

fieldsets：属性分组
fieldsets = [
    ('basic',{'fields': ['btitle']}),
    ('more', {'fields': ['bpub_date']}),
]

fields和fieldsets不能一起用
```
#### 关联对象
```
对于HeroInfo模型类，有两种注册方式

方式一：与BookInfo模型类相同
方式二：关联注册
按照BookInfor的注册方式完成HeroInfo的注册

接下来实现关联注册
from django.contrib import admin
from models import BookInfo,HeroInfo

class HeroInfoInline(admin.StackedInline):
    model = HeroInfo
    extra = 2 # 默认提供两个HeroInfo的位置

class BookInfoAdmin(admin.ModelAdmin):
    inlines = [HeroInfoInline]

admin.site.register(BookInfo, BookInfoAdmin)

可以将内嵌的方式改为表格
class HeroInfoInline(admin.TabularInline)
```
#### 布尔值的显示
```
发布性别的显示不是一个直观的结果，可以在HeroInfo模型类中定义方法进行封装
def gender(self):
    if self.hgender:
        return '男'
    else:
        return '女'
gender.short_description = '性别'


在admin注册中使用gender代替hgender
# admin.py
class HeroInfoAdmin(admin.ModelAdmin):
    list_display = ['id', 'hname', 'gender', 'hcontent']
```
### 视图
+ 在django中，视图对WEB请求进行回应
+ 视图接收reqeust对象作为第一个参数，包含了请求的信息
+ 视图就是一个Python函数，被定义在views.py中
```
from django.http import HttpResponse

def index(request):
    return HttpResponse('hello world!')
    
def detail(request,id):
    return HttpResponse("detail %s" % id)
    
定义完成视图后，需要配置urlconf，否则无法处理请求
```
### URLconf
+ 在Django中，定义URLconf包括正则表达式、视图两部分
+ Django使用正则表达式匹配请求的URL，一旦匹配成功，则调用应用的视图
+ 注意：只匹配路径部分，即除去域名、参数后的字符串
```
在test1/urls.py包含booktest.urls，使主urlconf连接到booktest.urls模块
urlpatterns = [
    url(r'^admin/', include(admin.site.urls)),
    url(r'^', include('booktest.urls')),
]
```
```
在booktest中创建的urls.py文件并添加urlconf
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^$', views.index),
    url(r'^([0-9]+)/$', views.detail),
]
```
### 模板
+ 模板是html页面，可以根据视图中传递的数据填充值
+ 创建模板的目录如下图：
![](http://p3ek8hcdl.bkt.clouddn.com/image/templates.png)
+ 修改settings.py文件，设置TEMPLATES的DIRS值
```
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```
+ 在模板中访问视图传递的数据
```
{{输出值，可以是变量，也可以是对象.属性}}
{%执行代码段%}
```
#### 定义index.html模板
```
<!DOCTYPE html>
<html>
<head>
  <title>首页</title>
</head>
<body>
<h1>图书列表</h1>
<ul>
{%for book in booklist%}
<li>
  <a href="{{book.id}}">
    {{book.btitle}}
  </a>
</li>
{%endfor%}
</ul>
</body>
</html>
```
#### 定义detail.html模板
在模板中访问对象成员时，都以属性的方式访问，即方法也不能加括号
```
<!DOCTYPE html>
<html>
<head>
  <title>详细页</title>
</head>
<body>
<h1>{{book.btitle}}</h1>
<ul>
  {%for hero in book.heroinfo_set.all%}
  <li>{{hero.hname}}---{{hero.hcontent}}</li>
  {%endfor%}
</ul>
</body>
</html>
```
### 使用模板
编辑views.py文件，在方法中调用模板
```
from django.http import HttpResponse
from django.template import RequestContext,loader
from .models import *


def index(request):
    booklist = BookInfo.objects.all()
    template = loader.get_template('booktest/index.html')
    context = RequestContext(request, {'booklist': booklist})
    return HttpResponse(template.render(context))

def detail(request, id):
    book = BookInfo.objects.get(pk=id)
    template = loader.get_template('booktest/detail.html')
    context = RequestContext(request, {'book': book})
    return HttpResponse(template.render(context))
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/templates.jpg)

### 去除模板的硬编码
+ 在index.html模板中，超链接是硬编码的，此时的请求地址为“127.0.0.1/1/”
```
<a href="{{book.id}}">
```
+ 看如下情况：将urlconf中详细页改为如下，链接就找不到了
```
url(r'^book/([0-9]+)/$', views.detail),
```
+ 此时的请求地址应该为“127.0.0.1/book/1/”
+ 问题总结：如果在模板中地址硬编码，将来urlconf修改后，地址将失效
+ 解决：使用命名的url设置超链接
+ 修改test1/urls.py文件，在include中设置namespace
```
url(r'^admin/', include(admin.site.urls, namespace='booktest')),
```
+ 修改booktest/urls.py文件，设置name
```
url(r'^book/([0-9]+)/$', views.detail, name="detail"),
```
+ 修改index.html模板中的链接
```
<a href="{%url 'booktest:detail' book.id%}">
```
后面章节中会详细说 反向解析
### Render简写
Django提供了函数Render()简化视图调用模板、构造上下文
```
from django.shortcuts import render
from .models import *


def index(request):
    booklist = BookInfo.objects.all()
    context = {'booklist': booklist}
    return render(request,'booktest/index.html', context)

def detail(request, id):
    book = BookInfo.objects.get(pk=id)
    context = {'book': book}
    return render(request, 'booktest/detail.html', context)
```
## 模型
### ORM简介
+ MVC框架中包括一个重要的部分，就是ORM，它实现了数据模型与数据库的解耦，即数据模型的设计不需要依赖于特定的数据库，通过简单的配置就可以轻松更换数据库
+ ORM是“对象-关系-映射”的简称，主要任务是：
    + 根据对象的类型生成表结构
    + 将对象、列表的操作，转换为sql语句
    + 将sql查询到的结果转换为对象、列表
+ 这极大的减轻了开发人员的工作量，不需要面对因数据库变更而导致的无效劳动
+ Django中的模型包含存储数据的字段和约束，对应着数据库中唯一的表
![](http://p3ek8hcdl.bkt.clouddn.com/image/orm.png)
### 使用MySql数据库
+ 在虚拟环境中安装mysql包
```
pip install mysql-python
```
+ 在mysql中创建数据库
```
create databases test1 charset=utf8
```
+ 打开settings.py文件，修改DATABASES项
```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'test1',
        'USER': '用户名',
        'PASSWORD': '密码',
        'HOST': '数据库服务器ip，本地可以使用localhost',
        'PORT': '端口，默认为3306',
    }
}
```
### 开发流程
1. 在models.py中定义模型类，要求继承自models.Model
2. 把应用加入settings.py文件的installed_app项
3. 生成迁移文件
4. 执行迁移生成表
5. 使用模型类进行crud操作

### 使用数据库生成模型类
```
python manage.py inspectdb > booktest/models.py
```
### 定义模型

#### 定义模型
+ 在模型中定义属性，会生成表中的字段
+ django根据属性的类型确定以下信息：
    + 当前选择的数据库支持字段的类型
    + 渲染管理表单时使用的默认html控件
    + 在管理站点最低限度的验证
+ django会为表增加自动增长的主键列，每个模型只能有一个主键列，如果使用选项设置某属性为主键列后，则django不会再生成默认的主键列。如果你想指定一个自定义主键字段，只要在某个字段上指定 primary_key=True 即可 例：id = models.AutoField(primary_key=True)
+ 属性命名限制
    + 不能是python的保留关键字
    + 由于django的查询方式，不允许使用连续的下划线
#### 定义属性
+ 定义属性时，需要字段类型
+ 字段类型被定义在django.db.models.fields目录下，为了方便使用，被导入到django.db.models中
+ 使用方式
    1. 导入from django.db import models
    2. 通过models.Field创建字段类型的对象，赋值给属性
+ 对于重要数据都做逻辑删除，不做物理删除，实现方法是定义isDelete属性，类型为BooleanField，默认值为False

#### 字段类型
+ AutoField：一个根据实际ID自动增长的IntegerField，通常不指定
    + 如果不指定，一个主键字段将自动添加到模型中
+ BooleanField：true/false 字段，此字段的默认表单控制是CheckboxInput
+ NullBooleanField：支持null、true、false三种值
+ CharField(max_length=字符长度)：字符串，默认的表单样式是 TextInput
+ TextField：大文本字段，一般超过4000使用，默认的表单控件是Textarea
+ IntegerField：整数
+ DecimalField(max_digits=None, decimal_places=None)：使用python的Decimal实例表示的十进制浮点数
    + DecimalField.max_digits：位数总数
    + DecimalField.decimal_places：小数点后的数字位数
+ FloatField：用Python的float实例来表示的浮点数
+ DateField[auto_now=False, auto_now_add=False])：使用Python的datetime.date实例表示的日期
    + 参数DateField.auto_now：每次保存对象时，自动设置该字段为当前时间，用于"最后一次修改"的时间戳，它总是使用当前日期，默认为false
    + 参数DateField.auto_now_add：当对象第一次被创建时自动设置当前时间，用于创建的时间戳，它总是使用当前日期，默认为false
    + 该字段默认对应的表单控件是一个TextInput. 在管理员站点添加了一个JavaScript写的日历控件，和一个“Today"的快捷按钮，包含了一个额外的invalid_date错误消息键
    + auto_now_add, auto_now, and default 这些设置是相互排斥的，他们之间的任何组合将会发生错误的结果
+ TimeField：使用Python的datetime.time实例表示的时间，参数同DateField
+ DateTimeField：使用Python的datetime.datetime实例表示的日期和时间，参数同DateField
+ FileField：一个上传文件的字段
+ ImageField：继承了FileField的所有属性和方法，但对上传的对象进行校验，确保它是个有效的image

#### 字段选项
+ 通过字段选项，可以实现对字段的约束
+ 在字段对象时通过关键字参数指定
+ null：如果为True，Django 将空值以NULL 存储到数据库中，默认值是 False
+ blank：如果为True，则该字段允许为空白，默认值是 False
+ 对比：null是数据库范畴的概念，blank是表单验证范畴的
+ db_column：字段的名称，如果未指定，则使用属性的名称
+ db_index：若值为 True, 则在表中会为此字段创建索引
+ default：默认值
+ primary_key：若为 True, 则该字段会成为模型的主键字段
+ unique：如果为 True, 这个字段在表中必须有唯一值

#### 关系
+ 关系的类型包括
    + ForeignKey：一对多，将字段定义在多的端中
    + ManyToManyField：多对多，将字段定义在两端中
    + OneToOneField：一对一，将字段定义在任意一端中
+ 可以维护递归的关联关系，使用'self'指定，详见“自关联”
+ 用一访问多：对象.模型类小写_set
```
bookinfo.heroinfo_set
```
+ 用一访问一：对象.模型类小写
```
heroinfo.bookinfo
```
+ 访问id：对象.属性_id
```
heroinfo.book_id
```
#### 元选项
+ 在模型类中定义类Meta，用于设置元信息
+ 元信息db_table：定义数据表名称，推荐使用小写字母，数据表的默认名称
```
<app_name>_<model_name> # 应用名称_模型类名
```
+ ordering：对象的默认排序字段，获取对象的列表时使用，接收属性构成的列表
```
class BookInfo(models.Model):
    ...
    class Meta():
        ordering = ['id']
        
字符串前加-表示倒序，不加-表示正序
class BookInfo(models.Model):
    ...
    class Meta():
        ordering = ['-id']

排序会增加数据库的开销!!!
```
#### 示例演示
+ 创建test2项目，并创建booktest应用，使用mysql数据库
+ 定义图书模型
```
class BookInfo(models.Model):
    btitle = models.CharField(max_length=20)
    bpub_date = models.DateTimeField()
    bread = models.IntegerField(default=0)
    bcommet = models.IntegerField(default=0)
    isDelete = models.BooleanField(default=False)
```
+ 英雄模型
```
class HeroInfo(models.Model):
    hname = models.CharField(max_length=20)
    hgender = models.BooleanField(default=True)
    hcontent = models.CharField(max_length=100)
    isDelete = models.BooleanField(default=False)
    hbook = models.ForeignKey('BookInfo')
```
+ 定义index、detail视图
+ index.html、detail.html模板
+ 配置url，能够完成图书及英雄的展示

#### 测试数据
+ 模型BookInfo的测试数据
```
insert into booktest_bookinfo(btitle,bpub_date,bread,bcommet,isDelete) values
('射雕英雄传','1980-5-1',12,34,0),
('天龙八部','1986-7-24',36,40,0),
('笑傲江湖','1995-12-24',20,80,0),
('雪山飞狐','1987-11-11',58,24,0);
```
+ 模型HeroInfo的测试数据
```
insert into booktest_heroinfo(hname,hgender,hbook_id,hcontent,isDelete) values
('郭靖',1,1,'降龙十八掌',0),
('黄蓉',0,1,'打狗棍法',0),
('黄药师',1,1,'弹指神通',0),
('欧阳锋',1,1,'蛤蟆功',0),
('梅超风',0,1,'九阴白骨爪',0),
('乔峰',1,2,'降龙十八掌',0),
('段誉',1,2,'六脉神剑',0),
('虚竹',1,2,'天山六阳掌',0),
('王语嫣',0,2,'神仙姐姐',0),
('令狐冲',1,3,'独孤九剑',0),
('任盈盈',0,3,'弹琴',0),
('岳不群',1,3,'华山剑法',0),
('东方不败',0,3,'葵花宝典',0),
('胡斐',1,4,'胡家刀法',0),
('苗若兰',0,4,'黄衣',0),
('程灵素',0,4,'医术',0),
('袁紫衣',0,4,'六合拳',0);
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/books.jpg)
 
![](http://p3ek8hcdl.bkt.clouddn.com/image/heros.jpg)

### 模型成员
#### 类的属性
+ objects：是Manager类型的对象，用于与数据库进行交互
+ 当定义模型类时没有指定管理器，则Django会为模型类提供一个名为objects的管理器
+ 支持明确指定模型类的管理器
```
class BookInfo(models.Model):
    ...
    books = models.Manager()
```
+ 当为模型类指定管理器后，django不再为模型类生成名为objects的默认管理器

#### 管理器Manager
+ 管理器是Django的模型进行数据库的查询操作的接口，Django应用的每个模型都拥有至少一个管理器
+ 自定义管理器类主要用于两种情况
+ 情况一：向管理器类中添加额外的方法：见下面“创建对象”中的方式二
+ 情况二：修改管理器返回的原始查询集：重写get_queryset()方法
```
class BookInfoManager(models.Manager):
    def get_queryset(self):
        return super(BookInfoManager, self).get_queryset().filter(isDelete=False)
class BookInfo(models.Model):
    ...
    books = BookInfoManager()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/manager.jpg)

#### 创建对象
+ 当创建对象时，django不会对数据库进行读写操作
+ 调用save()方法才与数据库交互，将对象保存到数据库中
+ 使用关键字参数构造模型对象很麻烦，推荐使用下面的两种之式
+ 说明： _init _方法已经在基类models.Model中使用，在自定义模型中无法使用
+ 方式一：在模型类中增加一个类方法
```
class BookInfo(models.Model):
    ...
    @classmethod
    def create(cls, title, pub_date):
        book = cls(btitle=title, bpub_date=pub_date)
        book.bread=0
        book.bcommet=0
        book.isDelete = False
        return book
        
调用：book=BookInfo.create("hello",datetime(1980,10,11));
保存：book.save()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/clscreate.jpg)

+ 方式二：在自定义管理器中添加一个方法
+ 在管理器的方法中，可以通过self.model来得到它所属的模型类
```
class BookInfoManager(models.Manager):
    def create_book(self, title, pub_date):
        book = self.model()
        book.btitle = title
        book.bpub_date = pub_date
        book.bread=0
        book.bcommet=0
        book.isDelete = False
        return book

class BookInfo(models.Model):
    ...
    books = BookInfoManager()
调用：book=BookInfo.books.create_book("abc",datetime(1980,1,1))
保存：book.save()
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/bookscreate.jpg)

#### 实例的属性
+ DoesNotExist：在进行单个查询时，模型的对象不存在时会引发此异常，结合try/except使用

#### 实例的方法
+ save()：将模型对象保存到数据表中
+ delete()：将模型对象从数据表中删除(物理删除)

### 模型查询
#### 简介
+ 查询集表示从数据库中获取的对象集合
+ 查询集可以含有零个、一个或多个过滤器
+ 过滤器基于所给的参数限制查询的结果
+ 从Sql的角度，查询集和select语句等价，过滤器像where和limit子句
+ 接下来主要讨论如下知识点
    + 查询集
    + 字段查询：比较运算符，F对象，Q对象

#### 查询集
+ 在管理器上调用过滤器方法会返回查询集
+ 查询集经过过滤器筛选后返回新的查询集，因此可以写成链式过滤
+ 惰性执行：创建查询集不会带来任何数据库的访问，直到调用数据时，才会访问数据库
+ 何时对查询集求值：迭代，序列化，与if合用
+ 返回查询集的方法，称为过滤器
    + all()
    + filter()返回一个新的QuerySet，包含与给定的查询参数匹配的对象。
    + exclude()返回一个新的QuerySet，它包含不满足给定的查找参数的对象。
    + order_by()排序
    + values()：一个对象构成一个字典，然后构成一个列表返回
+ 写法：
```
filter(键1=值1,键2=值2)
等价于
filter(键1=值1).filter(键2=值2)
```
![](http://p3ek8hcdl.bkt.clouddn.com/filter_exclude.jpg)
 
![](http://p3ek8hcdl.bkt.clouddn.com/orderby_values.jpg)

+ 返回单个值的方法
    + get()：返回单个满足条件的对象
        + 如果未找到会引发"模型类.DoesNotExist"异常
        + 如果多条被返回，会引发"模型类.MultipleObjectsReturned"异常
    + count()：返回当前查询的总条数
    + first()：返回第一个对象
    + last()：返回最后一个对象
    + exists()：判断查询集中是否有数据，如果有则返回True

![](http://p3ek8hcdl.bkt.clouddn.com/select.jpg)

#### 限制查询集
+ 查询集返回列表，可以使用下标的方式进行限制，等同于sql中的limit和offset子句
+ 注意：不支持负数索引
+ 使用下标后返回一个新的查询集，不会立即执行查询
+ 如果获取一个对象，直接使用[0]，等同于[0:1].get()，但是如果没有数据，[0]引发IndexError异常，[0:1].get()引发DoesNotExist异常

#### 查询集的缓存
+ 每个查询集都包含一个缓存来最小化对数据库的访问
+ 在新建的查询集中，缓存为空，首次对查询集求值时，会发生数据库查询，django会将查询的结果存在查询集的缓存中，并返回请求的结果，接下来对查询集求值将重用缓存的结果
+ 情况一：这构成了两个查询集，无法重用缓存，每次查询都会与数据库进行一次交互，增加了数据库的负载
```
print([e.title for e in Entry.objects.all()])
print([e.title for e in Entry.objects.all()])
```
+ 情况二：两次循环使用同一个查询集，第二次使用缓存中的数据
```
querylist=Entry.objects.all()
print([e.title for e in querylist])
print([e.title for e in querylist])
```
+ 何时查询集不会被缓存：当只对查询集的部分进行求值时会检查缓存，但是如果这部分不在缓存中，那么接下来查询返回的记录将不会被缓存，这意味着使用索引来限制查询集将不会填充缓存，如果这部分数据已经被缓存，则直接使用缓存中的数据
```
例如，重复获取查询集对象中一个特定的索引将每次都查询数据库：
>>> queryset = Entry.objects.all()
>>> print queryset[5] # Queries the database
>>> print queryset[5] # Queries the database again

然而，如果已经对全部查询集求值过，则将检查缓存：
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # Queries the database
>>> print queryset[5] # Uses cache
>>> print queryset[5] # Uses cache
```

#### 字段查询

+ 实现where子名，作为方法filter()、exclude()、get()的参数
+ 语法：属性名称__比较运算符=值
+ 表示两个下划线，左侧是属性名称，右侧是比较类型
+ 对于外键，使用“属性名_id”表示外键的原始值
+ 转义：like语句中使用了%与，匹配数据中的%与，在过滤器中直接写，例如：filter(title__contains="%")=>where title like '%\%%'，表示查找标题中包含%的

#### 比较运算符
+ exact：表示判等，大小写敏感；如果没有写“ 比较运算符”，表示判等
```
filter(isDelete=False)
```
+ contains：是否包含，大小写敏感
```
exclude(btitle__contains='传')
```
+ startswith、endswith：以value开头或结尾，大小写敏感
```
exclude(btitle__endswith='传')
```
+ isnull、isnotnull：是否为null
```
filter(btitle__isnull=False)
```
+ 在前面加个i表示不区分大小写，如iexact、icontains、istarswith、iendswith
+ in：是否包含在范围内
```
filter(pk__in=[1, 2, 3, 4, 5])
```
+ gt、gte、lt、lte：大于、大于等于、小于、小于等于
```
filter(id__gt=3)
```
+ year、month、day、week_day、hour、minute、second：对日期间类型的属性进行运算
```
filter(bpub_date__year=1980)
filter(bpub_date__gt=date(1980, 12, 31))
```
+ 跨关联关系的查询：处理join查询
    + 语法：模型类名 <属性名> <比较>
    + 注：可以没有__<比较>部分，表示等于，结果同inner join
    + 可返向使用，即在关联的两个模型中都可以使用
```
filter(heroinfo__hcontent__contains='八')
```
+ 查询的快捷方式：pk，pk表示primary key，默认的主键是id
```
filter(pk__lt=6)
```
#### 聚合函数
+ 使用aggregate()函数返回聚合函数的值
+ 函数：Avg，Count，Max，Min，Sum
```
from django.db.models import Max
maxDate = list.aggregate(Max('bpub_date'))
```
+ count的一般用法：
```
count = BookInfo.objects.count()
```

#### F对象
+ 可以使用模型的字段A与字段B进行比较，如果A写在了等号的左边，则B出现在等号的右边，需要通过F对象构造
```
list.filter(bread__gte=F('bcommet'))
```
+ django支持对F()对象使用算数运算
```
list.filter(bread__gte=F('bcommet') * 2)
```
+ F()对象中还可以写作“模型类__列名”进行关联查询
```
list.filter(isDelete=F('heroinfo__isDelete'))
```
+ 对于date/time字段，可与timedelta()进行运算
```
list.filter(bpub_date__lt=F('bpub_date') + timedelta(days=1))
```
#### Q对象
+ 过滤器的方法中关键字参数查询，会合并为And进行
+ 需要进行or查询，使用Q()对象
+ Q对象(django.db.models.Q)用于封装一组关键字参数，这些关键字参数与“比较运算符”中的相同
```
from django.db.models import Q
list.filter(Q(pk__lt=6))
```
+ Q对象可以使用&（and）、|（or）操作符组合起来
+ 当操作符应用在两个Q对象时，会产生一个新的Q对象
```
list.filter(pk__lt=6).filter(bcommet__gt=10)
list.filter(Q(pk__lt=6) | Q(bcommet__gt=10)) # 主键小于6或者评论大于10的
```
+ 使用~（not）操作符在Q对象前表示取反
```
list.filter(~Q(pk__lt=6)) # 主键不小于6的，即 大于等于6
```
+ 可以使用&|~结合括号进行分组，构造做生意复杂的Q对象
+ 过滤器函数可以传递一个或多个Q对象作为位置参数，如果有多个Q对象，这些参数的逻辑为and
+ 过滤器函数可以混合使用Q对象和关键字参数，所有参数都将and在一起，Q对象必须位于关键字参数的前面

### 自连接
+ 对于地区信息，属于一对多关系，使用一张表，存储所有的信息
+ 类似的表结构还应用于分类信息，可以实现无限级分类
+ 新建模型AreaInfo，生成迁移
```
class AreaInfo(models.Model):
    atitle = models.CharField(max_length=20)
    aParent = models.ForeignKey('self', null=True, blank=True)
```
+ 访问关联对象
```
上级对象：area.aParent
下级对象：area.areainfo_set.all()
```
+ [加入测试数据](http://p3ek8hcdl.bkt.clouddn.com/txt/area.txt)
+ 在booktest/views.py中定义视图area
```
from models import AreaInfo
def area(request):
    area = AreaInfo.objects.get(pk=130100)
    return render(request, 'booktest/area.html', {'area': area})
```
+ 定义模板area.html
```
<!DOCTYPE html>
<html>
<head>
    <title>地区</title>
</head>
<body>
当前地区：{{area.atitle}}
<hr/>
上级地区：{{area.aParent.atitle}}
<hr/>
下级地区：
<ul>
    { %for a in area.areainfo_set.all%}
    <li>{{a.atitle}}</li>
    { %endfor%}
</ul>
</body>
</html>
```
+ 在booktest/urls.py中配置一个新的urlconf
```
urlpatterns = [
    url(r'^area/$', views.area, name='area')
]
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/area.jpg)

## 视图
+ 视图接受Web请求并且返回Web响应
+ 视图就是一个python函数，被定义在views.py中
+ 响应可以是一张网页的HTML内容，一个重定向，一个404错误等等
+ 响应处理过程如下图：

![](http://p3ek8hcdl.bkt.clouddn.com/image/views.jpg)

### URLconf
+ 在settings.py文件中通过ROOT_URLCONF指定根级url的配置
+ urlpatterns是一个url()实例的列表
+ 一个url()对象包括：
    + 正则表达式
    + 视图函数
    + 名称name
+ 编写URLconf的注意：
    + 若要从url中捕获一个值，需要在它周围设置一对圆括号
    + 不需要添加一个前导的反斜杠，如应该写作'test/'，而不应该写作'/test/'
    + 每个正则表达式前面的r表示字符串不转义
+ 请求的url被看做是一个普通的python字符串，进行匹配时不包括get或post请求的参数及域名
```
http://www.ixysec.com/python/django/?i=1&p=new，只匹配“python/django/”部分
```
+ 正则表达式非命名组，通过位置参数传递给视图
![](http://p3ek8hcdl.bkt.clouddn.com/image/viewstest.jpg)

![](http://p3ek8hcdl.bkt.clouddn.com/viewstest2.jpg)

+ 正则表达式命名组，通过关键字参数传递给视图，本例中关键字参数为id
```
url(r'^(?P<id1>\d+)/(?P<id3>\d+)/(?P<id2>\d+)/$', views.test),
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/viewstest3.jpg)

+ 参数匹配规则：优先使用命名参数，如果没有命名参数则使用位置参数
+ 每个捕获的参数都作为一个普通的python字符串传递给视图
+ 性能：urlpatterns中的每个正则表达式在第一次访问它们时被编译，这使得系统相当快

#### 包含其它的URLconfs
+ 在应用中创建urls.py文件，定义本应用中的urlconf，再在项目的settings中使用include()
```
from django.conf.urls import include, url
urlpatterns = [
    url(r'^', include('booktest.urls', namespace='booktest')),
]
```
+ 匹配过程：先与主URLconf匹配，成功后再用剩余的部分与应用中的URLconf匹配
```
请求http://www.ixysec.com/booktest/1/
在sesstings.py中的配置：
url(r'^booktest/', include('booktest.urls', namespace='booktest')),
在booktest应用urls.py中的配置
url(r'^(\d+)/$', views.detail, name='detail'),
匹配部分是：/booktest/1/
匹配过程：在settings.py中与“booktest/”成功，再用“1/”与booktest应用的urls匹配
```
+ 使用include可以去除urlconf的冗余
+ 参数：视图会收到来自父URLconf、当前URLconf捕获的所有参数
+ 在include中通过namespace定义命名空间，用于反解析

#### URL的反向解析
+ 如果在视图、模板中使用硬编码的链接，在urlconf发生改变时，维护是一件非常麻烦的事情
+ 解决：在做链接时，通过指向urlconf的名称，动态生成链接地址
+ 视图：使用django.core.urlresolvers.reverse()函数
+ 模板：使用url模板标签
```
<a href="{%url 'namespace:name'%}">
```
### 视图函数
#### 定义视图
+ 本质就是一个函数
+ 视图的参数
    + 一个HttpRequest实例
    + 通过正则表达式组获取的位置参数
    + 通过正则表达式组获得的关键字参数
+ 在应用目录下默认有views.py文件，一般视图都定义在这个文件中
+ 如果处理功能过多，可以将函数定义到不同的py文件中

#### 404 (page not found) 视图
+ defaults.page_not_found(request, template_name='404.html')
+ 默认的404视图将传递一个变量给模板：request_path，它是导致错误的URL
+ 如果Django在检测URLconf中的每个正则表达式后没有找到匹配的内容也将调用404视图
+ 如果在settings中DEBUG设置为True，那么将永远不会调用404视图，而是显示URLconf 并带有一些调试信息
+ 在templates中创建404.html
```
<!DOCTYPE html>
<html>
<head>
    <title></title>
</head>
<body>
找不到了
<hr/>
{{request_path}}
</body>
</html>
```
+ 在settings.py中修改调试
```
DEBUG = False
ALLOWED_HOSTS = ['*', ]
```
+ 请求一个不存在的地址

![](http://p3ek8hcdl.bkt.clouddn.com/image/views404.jpg)

#### 500 (server error) 视图
+ defaults.server_error(request, template_name='500.html')
+ 在视图代码中出现运行时错误
+ 默认的500视图不会传递变量给500.html模板
+ 如果在settings中DEBUG设置为True，那么将永远不会调用505视图，而是显示URLconf 并带有一些调试信息

#### 400 (bad request) 视图
+ defaults.bad_request(request, template_name='400.html')
+ 错误来自客户端的操作
+ 当用户进行的操作在安全方面可疑的时候，例如篡改会话cookie

### Reqeust对象
#### HttpReqeust对象
+ 服务器接收到http协议的请求后，会根据报文创建HttpRequest对象
+ 视图函数的第一个参数是HttpRequest对象
+ 在django.http模块中定义了HttpRequest对象的API

#### 属性
+ 下面除非特别说明，属性都是只读的
+ path：一个字符串，表示请求的页面的完整路径，不包含域名
+ method：一个字符串，表示请求使用的HTTP方法，常用值包括：'GET'、'POST'
+ encoding：一个字符串，表示提交的数据的编码方式
    + 如果为None则表示使用浏览器的默认设置，一般为utf-8
    + 这个属性是可写的，可以通过修改它来修改访问表单数据使用的编码，接下来对属性的任何访问将使用新的encoding值
+ GET：一个类似于字典的对象，包含get请求方式的所有参数
+ POST：一个类似于字典的对象，包含post请求方式的所有参数
+ FILES：一个类似于字典的对象，包含所有的上传文件
+ COOKIES：一个标准的Python字典，包含所有的cookie，键和值都为字符串
+ session：一个既可读又可写的类似于字典的对象，表示当前的会话，只有当Django 启用会话的支持时才可用，详细内容见“状态保持”

#### 方法
+ is_ajax()：如果请求是通过XMLHttpRequest发起的，则返回True

#### QueryDict对象
+ 定义在django.http.QueryDict
+ request对象的属性GET、POST都是QueryDict类型的对象
+ 与python字典不同，QueryDict类型的对象用来处理同一个键带有多个值的情况
+ 方法get()：根据键获取值
    + 只能获取键的一个值
    + 如果一个键同时拥有多个值，获取最后一个值
```
dict.get('键',default)
或简写为
dict['键'] -> 如果没有那个键，则会报异常。推荐使用第一种
```
+ 方法getlist()：根据键获取值
    + 将键的值以列表返回，可以获取一个键的多个值
```
dict.getlist('键',default)
```
#### GET属性
+ QueryDict类型的对象
+ 包含get请求方式的所有参数
+ 与url请求地址中的参数对应，位于?后面
+ 参数的格式是键值对，如key1=value1
+ 多个参数之间，使用&连接，如key1=value1&key2=value2
+ 键是开发人员定下来的，值是可变的
+ 示例如下
+ 创建视图getTest1用于定义链接，getTest2用于接收一键一值，getTest3用于接收一键多值
```
def getTest1(request):
    return render(request,'booktest/getTest1.html')
def getTest2(request):
    return render(request,'booktest/getTest2.html')
def getTest3(request):
    return render(request,'booktest/getTest3.html')
```
+ 配置url
```
url(r'^getTest1/$', views.getTest1),
url(r'^getTest2/$', views.getTest2),
url(r'^getTest3/$', views.getTest3),
```
+ 创建getTest1.html，定义链接
```
<html>
<head>
    <title>Title</title>
</head>
<body>
链接1：一个键传递一个值
<a href="/getTest2/?a=1&b=2">gettest2</a><br>
链接2：一个键传递多个值
<a href="/getTest3/?a=1&a=2&b=3">gettest3</a>
</body>
</html>
```
+ 完善视图getTest2的代码
```
def getTest2(request):
    a=request.GET['a']
    b=request.GET['b']
    context={'a':a,'b':b}
    return render(request,'booktest/getTest2.html',context)
```
+ 创建getTest2.html，显示接收结果
```
<html>
<head>
    <title>Title</title>
</head>
<body>
a:{{ a }}<br>
b:{{ b }}
</body>
</html>
```
+ 完善视图getTest3的代码
```
def getTest3(request):
    a=request.GET.getlist('a')
    b=request.GET['b']
    context={'a':a,'b':b}
    return render(request,'booktest/getTest3.html',context)
```
+ 创建getTest3.html，显示接收结果
```
<html>
<head>
    <title>Title</title>
</head>
<body>
a:{% for item in a %}
{{ item }}
{% endfor %}
<br>
b:{{ b }}
</body>
</html>
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/gettest.jpg)

#### POST属性
+ QueryDict类型的对象
+ 包含post请求方式的所有参数
+ 与form表单中的控件对应
+ 控件要有name属性，则name属性的值为键，value属性的值为键，构成键值对提交
    + 对于checkbox控件，name属性一样为一组，当控件被选中后会被提交，存在一键多值的情况
+ 示例如下
+ 定义视图postTest1
```
def postTest1(request):
    return render(request,'booktest/postTest1.html')
```
+ 配置url
```
url(r'^postTest1/$',views.postTest1)
```
+ 创建模板postTest1.html
```
<html>
<head>
    <title>Title</title>
</head>
<body>
<form method="post" action="/postTest2/">
    姓名：<input type="text" name="uname"/><br>
    密码：<input type="password" name="upwd"/><br>
    性别：<input type="radio" name="ugender" value="1"/>男
    <input type="radio" name="ugender" value="0"/>女<br>
    爱好：<input type="checkbox" name="uhobby" value="胸口碎大石"/>胸口碎大石
    <input type="checkbox" name="uhobby" value="跳楼"/>跳楼
    <input type="checkbox" name="uhobby" value="喝酒"/>喝酒
    <input type="checkbox" name="uhobby" value="爬山"/>爬山<br>
    <input type="submit" value="提交"/>
</form>
</body>
</html>
```
+ 创建视图postTest2接收请求的数据
```
def postTest2(request):
    uname=request.POST['uname']
    upwd=request.POST['upwd']
    ugender=request.POST['ugender']
    uhobby=request.POST.getlist('uhobby')
    context={'uname':uname,'upwd':upwd,'ugender':ugender,'uhobby':uhobby}
    return render(request,'booktest/postTest2.html',context)
```
+ 配置url
```
url(r'^postTest2/$',views.postTest2)
```
+ 创建模板postTest2.html
```
<html>
<head>
    <title>Title</title>
</head>
<body>
{{ uname }}<br>
{{ upwd }}<br>
{{ ugender }}<br>
{{ uhobby }}
</body>
</html>
```
+ 注意：使用表单提交，注释掉settings.py中的中间件crsf
![](http://p3ek8hcdl.bkt.clouddn.com/image/jdfw.gif)

### Response对象
#### HttpResponse对象
+ 在django.http模块中定义了HttpResponse对象的API
+ HttpRequest对象由Django自动创建，HttpResponse对象由程序员创建
+ 不调用模板，直接返回数据
```
#coding=utf-8
from django.http import HttpResponse

def index(request):
    return HttpResponse('你好')
```
+ 调用模板
```
from django.http import HttpResponse
from django.template import RequestContext, loader

def index(request):
    t1 = loader.get_template('polls/index.html')
    context = RequestContext(request, {'h1': 'hello'})
    return HttpResponse(t1.render(context))
```
#### 方法
+ write(content)：以文件的方式写
+ flush()：以文件的方式输出缓存区
+ set_cookie(key, value='', max_age=None, expires=None, path='/', domain=None, secure=None, httponly=False)：设置Cookie
    + key、value都是字符串类型
    + max_age是一个整数，表示在指定秒数后过期
    + expires是一个datetime或timedelta对象，会话将在这个指定的日期/时间过期，注意datetime和timedelta值只有在使用PickleSerializer时才可序列化
    + max_age与expires二选一
    + 如果不指定过期时间，则两个星期后过期
    + 如果你想设置一个跨域的Cookie，请使用domain 参数。例如，domain=".lawrence.com" 将设置一个www.lawrence.com、blogs.lawrence.com 和calendars.lawrence.com 都可读的Cookie。否则，Cookie 将只能被设置它的域读取。
    + 如果你想阻止客服端的JavaScript 访问Cookie，可以设置httponly=True。
+ delete_cookie(key)：删除指定的key的Cookie，如果key不存在则什么也不发生
```
def cookieTest(reqeust):
    response = HttpResponse('hello')
    cookie = reqeust.COOKIES
    if 'blog' in cookie.keys():
        response.write('<h1>' + reqeust.COOKIES['blog'] + '</h1>')
    # response.set_cookie('blog', 'ixysec.com', 60*10)
    return response
```
#### 子类HttpResponseRedirect
+ Django包含了一系列的HttpResponse衍生类（子类），用来处理不同类型的HTTP 响应（response）。与 HttpResponse相同, 这些衍生类（子类）存在于django.http之中。
+ 重定向，服务器端跳转
+ 构造函数的第一个参数用来指定重定向的地址
```
from django.http import HttpResponse,HttpResponseRedirect

def redirect1(request):
    return HttpResponseRedirect('/redirect2/')

def redirect2(request):
    return HttpResponse('跳转后的地址')

```
![](http://p3ek8hcdl.bkt.clouddn.com/image/redirect.gif)

#### 子类JsonResponse
+ 返回json数据，一般用于异步请求
+ _init _(data)
+ 帮助用户创建JSON编码的响应
+ 参数data是字典对象
+ JsonResponse的默认Content-Type为application/json
```
from django.http import JsonResponse

def jsonTest(request):
    return JsonResponse({'data':[1,2,3,4]})
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/jsontest.jpg)

#### 简写函数
##### render
+ render(request, template_name[, context])
+ 结合一个给定的模板和一个给定的上下文字典，并返回一个渲染后的HttpResponse对象
+ request：该request用于生成response
+ template_name：要使用的模板的完整名称
+ context：添加到模板上下文的一个字典，视图将在渲染模板之前调用它
```
from django.shortcuts import render

def index(request):
    return render(request, 'booktest/index.html', {'h1': 'hello'})
```
##### 重定向
+ redirect(to)
+ 为传递进来的参数返回HttpResponseRedirect
+ to推荐使用反向解析
```
from django.shortcuts import redirect
from django.core.urlresolvers import reverse

def index(request):
    return redirect(reverse('booktest:index2'))
```

### 状态保持
+ http协议是无状态的：每次请求都是一次新的请求，不会记得之前通信的状态
+ 客户端与服务器端的一次通信，就是一次会话
+ 实现状态保持的方式：在客户端或服务器端存储与会话有关的数据
+ 存储方式包括cookie、session，会话一般指session对象
+ 使用cookie，所有数据存储在客户端，注意不要存储敏感信息
+ 推荐使用sesison方式，所有数据存储在服务器端，在客户端cookie中存储session_id ->session依赖于cookie
+ 状态保持的目的是在一段时间内跟踪请求者的状态，可以实现跨页面访问当前请求者的数据
+ 注意：不同的请求者之间不会共享这个数据，与请求者一一对应

#### 启用session
+ 使用django-admin startproject创建的项目默认启用
+ 在settings.py文件中
```
INSTALLED_APPS列表中添加：
'django.contrib.sessions',

MIDDLEWARE_CLASSES列表中添加：
'django.contrib.sessions.middleware.SessionMiddleware',
```
+ 禁用会话：删除上面指定的两个值，禁用会话将节省一些性能消耗

#### 使用session
+ 启用会话后，每个HttpRequest对象将具有一个session属性，它是一个类字典对象
+ get(key, default=None)：根据键获取会话的值
+ clear()：清除所有会话
+ flush()：删除当前的会话数据并删除会话的Cookie
+ del request.session['member_id']：删除会话

#### 用户登录示例
+ 操作效果如下图：

![](http://p3ek8hcdl.bkt.clouddn.com/image/session.gif)

+ 在views.py文件中创建视图
```
from django.shortcuts import render, redirect

def session(request):
    if 'uname' in request.session.keys():
        uname = request.session.get('uname')
    else:
        uname = '未登录'
    return render(request, 'booktest/session.html', {'uname': uname})

def session_login(request):
    return render(request, 'booktest/session_login.html')

def session_login_handle(request):
    request.session['uname'] = request.POST['uname']
    return redirect('/session/')

def session_logout(request):
    del request.session['uname']
    return redirect('/session/')
```
+ 配置url
```
urlpatterns = [
    url(r'^session/$', views.session),
    url(r'^session_login/$', views.session_login),
    url(r'^session_login_handle/$', views.session_login_handle),
    url(r'^session_logout/$', views.session_logout),
]
```
+ 创建模板session.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>你好，{{uname}}</h1>
<hr>
<a href="/session_login/">登陆</a>
<a href="/session_logout/">退出</a>

</body>
</html>
```
+ 创建模板session_login.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/session_login_handle/" method="post">
    {%csrf_token%}
    <input type="text" name="uname">
    <input type="submit" value="登陆">
</form>
</body>
</html>
```
#### 会话过期时间
+ set_expiry(value)：设置会话的超时时间
+ 如果没有指定，则两个星期后过期
+ 如果value是一个整数，会话将在values秒没有活动后过期
+ 若果value是一个imedelta对象，会话将在当前时间加上这个指定的日期/时间过期
+ 如果value为0，那么用户会话的Cookie将在用户的浏览器关闭时过期
+ 如果value为None，那么会话永不过期
+ 修改视图中session_login_handle函数，查看效果

![](http://p3ek8hcdl.bkt.clouddn.com/image/sessiontest.jpg)

#### 存储session
+ 使用存储会话的方式，可以使用settings.py的SESSION_ENGINE项指定
+ 基于数据库的会话：这是django默认的会话存储方式，需要添加django.contrib.sessions到的INSTALLED_APPS设置中，运行manage.py migrate在数据库中创建对应的session表，可显示指定为
```
SESSION_ENGINE='django.contrib.sessions.backends.db'
```
+ 基于缓存的会话：只存在本地内在中，如果丢失则不能找回，比数据库的方式读写更快
```
SESSION_ENGINE='django.contrib.sessions.backends.cache'
```
+ 可以将缓存和数据库同时使用：优先从本地缓存中获取，如果没有则从数据库中获取
```
SESSION_ENGINE='django.contrib.sessions.backends.cached_db'
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sessiondb.jpg)

![](http://p3ek8hcdl.bkt.clouddn.com/image/sessionbase64.jpg)

#### 使用Redis缓存session
+ 会话还支持文件、纯cookie、Memcached、Redis等方式存储，下面演示使用redis存储
+ 安装包
```
pip install django-redis-sessions
```
+ 修改settings中的配置，增加如下项
```
# 错误的配置 不会产生任何效果 
# 默认就是连接的localhost 之前在ubuntu上测试成功的 所以没有察觉
# 在windows连接ubuntu的redis一直失败 试了好久才发现这种方式根本没卵用。。。
# SESSION_REDIS_HOST = 'localhost'
# SESSION_REDIS_PORT = 6379
# SESSION_REDIS_DB = 0
# SESSION_REDIS_PASSWORD = ''
# SESSION_REDIS_PREFIX = 'session'

# 下面是正确的配置
SESSION_ENGINE = 'redis_sessions.session'
SESSION_REDIS = {
    'host': '192.168.1.102',
    'port': 6379,
    'db': 0,
    'password': '',
    'prefix': 'session',
    'socket_timeout': 1
}

服务器需要装有redis
```
+ 管理redis的命令
```
启动：sudo redis-server /etc/redis/redis.conf
停止：sudo redis-server stop
重启：sudo redis-server restart

redis-cli：使用客户端连接服务器
keys *：查看所有的键
get name：获取指定键的值
del name：删除指定名称的键
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/sessionredis.png)

## 模板
### 模板介绍
+ 作为Web框架，Django提供了模板，可以很便利的动态生成HTML
+ 模版系统致力于表达外观，而不是程序逻辑
+ 模板的设计实现了业务逻辑(view)与显示内容（template）的分离，一个视图可以使用任意一个模板，一个模板可以供多个视图使用
+ 模板包含
    + HTML的静态部分
    + 动态插入内容部分
+ Django模板语言，简写DTL，定义在django.template包中
+ 由startproject命令生成的settings.py定义关于模板的值：
    + DIRS定义了一个目录列表，模板引擎按列表顺序搜索这些目录以查找模板源文件
    + APP_DIRS告诉模板引擎是否应该在每个已安装的应用中查找模板
+ 常用方式：在项目的根目录下创建templates目录，设置DIRS值
```
DIRS=[os.path.join(BASE_DIR,"templates")]
```
#### 模板处理
+ Django处理模板分为两个阶段
+ Step1 加载：根据给定的标识找到模板然后预处理，通常会将它编译好放在内存中
```
loader.get_template(template_name)，返回一个Template对象
```
+ Step2 渲染：使用Context数据对模板插值并返回生成的字符串
```
Template对象的render(RequestContext)方法，使用context渲染模板
```
+ 加载渲染完整代码：
```
from django.template import loader, RequestContext
from django.http import HttpResponse

def index(request):
    tem = loader.get_template('temtest/index.html')
    context = RequestContext(request, {})
    return HttpResponse(tem.render(context))
```
#### 快捷函数
+ 为了减少加载模板、渲染模板的重复代码，django提供了快捷函数
+ render(request,'模板',context)
```
from django.shortcuts import render

def index(request):
    return render(request, 'temtest/index.html')
```
### 定义模板
+ 模板语言包括
    + 变量
    + 标签 { % 代码块 % }
    + 过滤器
    + 注释{# 代码或html #}

#### 变量
+ 语法：
```
{{ variable }}
```
+ 当模版引擎遇到一个变量，将计算这个变量，然后将结果输出
+ 变量名必须由字母、数字、下划线（不能以下划线开头）和点组成
+ 当模版引擎遇到点(".")，会按照下列顺序查询：
    1. 字典查询，例如：foo["bar"]
    2. 属性或方法查询，例如：foo.bar
    3. 数字索引查询，例如：foo[bar]
+ 如果变量不存在， 模版系统将插入'' (空字符串)
+ 在模板中调用方法时不能传递参数

##### 在模板中调用对象的方法
+ 在models.py中定义类HeroInfo
```
from django.db import models

class HeroInfo(models.Model):
    ...
    def showName(self):
        return self.hname
```
+ 在views.py中传递HeroInfo对象
```
from django.shortcuts import render
from models import *

def templatesTest(request):
    hero = HeroInfo(hname='ixysec')
    context = {'hero':hero}
    return render(request, 'booktest/templatesTest.html', context)

```
+ 在模板templatesTest.html中调用
```
{{hero.showName}}
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/templatestest.jpg)
#### 标签
+ 语法：{ % tag % }
+ 作用
    + 在输出中创建文本
    + 控制循环或逻辑
    + 加载外部信息到模板中供以后的变量使用
+ for标签
```
{ %for ... in ...%}
循环逻辑
{{forloop.counter}}表示当前是第几次循环
{ %empty%}
给出的列表为或列表不存在时，执行此处
{ %endfor%}
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/forTest.jpg)
+ if标签
```
{ %if ...%}
逻辑1
{ %elif ...%}
逻辑2
{ %else%}
逻辑3
{ %endif%}
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/iftest.jpg)
+ comment标签
```
{ % comment % }
多行注释
{ % endcomment % }
```
+ include：加载模板并以标签内的参数渲染
```
{%include "foo/bar.html" %}
```
+ url：反向解析
```
{% url 'name' p1 p2 %}
```
+ csrf_token：这个标签用于跨站请求伪造保护
```
{% csrf_token %}
```
+ 布尔标签：and、or，and比or的优先级高
+ block、extends：详见“模板继承”
+ autoescape：详见“HTML转义”

#### 过滤器
+ 语法：{ { 变量|过滤器 }}，例如{ { name|lower }}，表示将变量name的值变为小写输出
+ 使用管道符号 (|)来应用过滤器
+ 通过使用过滤器来改变变量的计算结果
+ 可以在if标签中使用过滤器结合运算符
```
if list1|length > 1
```
+ 过滤器能够被“串联”，构成过滤器链
```
name|lower|upper
```
+ 过滤器可以传递参数，参数使用引号包起来
```
list|join:","
如果list是列表['a'， 'b'， 'c'] ，输出将是字符串'a,b,c'。
```
+ default：如果一个变量没有被提供，或者值为false或空，则使用默认值，否则使用变量的值
```
value|default:"什么也没有"
```
+ date：根据给定格式对一个date变量格式化
```
value|date:'Y-m-d'
```
[过滤器](http://python.usyiyi.cn/documents/django_182/ref/templates/builtins.html)

#### 注释
+ 单行注释
```
{#...#}
```
+ 注释可以包含任何模版代码，有效的或者无效的都可以
```
{# { % if foo % }bar{ % else % } #}
```
+ 使用comment标签注释模版中的多行内容

### 反向解析
```
返回一个绝对路径的引用(不包含域名的URL)，该引用匹配一个给定的视图函数和一些可选的参数。
它可以避免在模板中硬编码链接路径。
{% url 'myapp:view-name' p1 p2 %}
警告
不要忘记在函数路径或模式名称周围加引号，否则值将被解释为上下文变量！
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/urltest.png)

![](http://p3ek8hcdl.bkt.clouddn.com/image/urltest1.jpg)

![](http://p3ek8hcdl.bkt.clouddn.com/image/urltest2.jpg)

![](http://p3ek8hcdl.bkt.clouddn.com/image/urltest3.jpg)

### 模板继承

+ 模板继承可以减少页面内容的重复定义，实现页面内容的重用
+ 典型应用：网站的头部、尾部是一样的，这些内容可以定义在父模板中，子模板不需要重复定义
+ block标签：在父模板中预留区域，在子模板中填充
+ extends继承：继承，写在模板文件的第一行
+ 定义父模板base.html
```
{ %block block_name%}
这里可以定义默认值
如果不定义默认值，则表示空字符串
{ %endblock%}
```
+ 定义子模板index.html
```
{ % extends "base.html" %}
```
+ 在子模板中使用block填充预留区域
```
{ %block block_name%}
实际填充内容
{ %endblock%}
```
#### 说明
+ 如果在模版中使用extends标签，它必须是模版中的第一个标签
+ 不能在一个模版中定义多个相同名字的block标签
+ 子模版不必定义全部父模版中的blocks，如果子模版没有定义block，则使用了父模版中的默认值
+ 如果发现在模板中大量的复制内容，那就应该把内容移动到父模板中
+ 使用可以获取父模板中block的内容
+ 为了更好的可读性，可以给endblock标签一个名字
```
{ % block block_name %}
区域内容
{ % endblock block_name %}
```
### HTML转义
+ Django对字符串进行自动HTML转义，如在模板中输出如下值：
```
视图代码：
def index(request):
    return render(request, 'temtest/index2.html',
                  {
                      't1': '<h1>hello</h1>'
                  })
模板代码：
{{t1}}
```
+ 显示效果如下：
```
<h1>hello</h1>
```
#### 会被自动转义的字符
+ html转义，就是将包含的html标签输出，而不被解释执行，原因是当显示用户提交字符串时，可能包含一些攻击性的代码，如js脚本
+ Django会将如下字符自动转义：
```
< 会转换为&lt;

> 会转换为&gt;

' (单引号) 会转换为&#39;

" (双引号)会转换为 &quot;

& 会转换为 &amp;
```
+ 当显示不被信任的变量时使用escape过滤器，一般省略，因为Django自动转义
```
{{t1|escape}}
```
#### 关闭转义
+ 对于变量使用safe过滤器
```
{{ data|safe }}
```
+ 对于代码块使用autoescape标签
```
{ % autoescape off %}
{{ body }}
{ % endautoescape %}
```
+ 标签autoescape接受on或者off参数
+ 自动转义标签在base模板中关闭，在child模板中也是关闭的

#### 手动转义
```
{ { data|default:"<b>123</b>" }}
应写为
{ { data|default:"&lt;b&gt;123&lt;/b&gt;" }}
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/htmltest.jpg)

### csrf
+ 全称Cross Site Request Forgery，跨站请求伪造
+ 某些恶意网站上包含链接、表单按钮或者JavaScript，它们会利用登录过的用户在浏览器中的认证信息试图在你的网站上完成某些操作，这就是跨站攻击
+ 演示csrf如下
+ 创建视图csrf1用于展示表单，csrf2用于接收post请求
```
def csrf1(request):
    return render(request,'booktest/csrf1.html')
def csrf2(request):
    uname=request.POST['uname']
    return render(request,'booktest/csrf2.html',{'uname':uname})
```
+ 配置url
```
url(r'^csrf1/$', views.csrf1),
url(r'^csrf2/$', views.csrf2),
```
+ 创建模板csrf1.html用于展示表单
```
<html>
<head>
    <title>Title</title>
</head>
<body>
<form method="post" action="/crsf2/">
    <input name="uname"><br>
    <input type="submit" value="提交"/>
</form>
</body>
</html>
```
+ 创建模板csrf2用于展示接收的结果
```
<html>
<head>
    <title>Title</title>
</head>
<body>
{{ uname }}
</body>
</html>
```
+ 在浏览器中访问，查看效果，报错如下：
![](http://p3ek8hcdl.bkt.clouddn.com/image/csrf1.png)

+ 将settings.py中的中间件代码'django.middleware.csrf.CsrfViewMiddleware'注释
+ 查看csrf1的源代码，复制，在自己的网站内建一个html文件，粘贴源码，访问查看效果

#### 防csrf的使用
+ 在django的模板中，提供了防止跨站请求伪造攻击的方法，使用步骤如下：
+ step1：在settings.py中启用'django.middleware.csrf.CsrfViewMiddleware'中间件，此项在创建项目时，默认被启用
+ step2：在csrf1.html中添加标签
```
<form>
{% csrf_token %}
...
</form>
```
+ step3：测试刚才的两个请求，发现跨站的请求被拒绝了，效果如下图

![](http://p3ek8hcdl.bkt.clouddn.com/image/csrf2.png)
#### 取消保护
+ 如果某些视图不需要保护，可以使用装饰器csrf_exempt，模板中也不需要写标签，修改csrf2的视图如下
```
from django.views.decorators.csrf import csrf_exempt

@csrf_exempt
def csrf2(request):
    uname=request.POST['uname']
    return render(request,'booktest/csrf2.html',{'uname':uname})
```
+ 运行上面的两个请求，发现都可以请求

#### 保护原理
+ 加入标签后，可以查看源代码，发现多了如下代码
```
<input type='hidden' name='csrfmiddlewaretoken' value='nGjAB3Md9ZSb4NmG1sXDolPmh3bR2g59' />
```
+ 在浏览器的调试工具中，通过network标签可以查看cookie信息
+ 本站中自动添加了cookie信息，如下图

![](http://p3ek8hcdl.bkt.clouddn.com/image/csrf3.png)
+ 查看跨站的信息，并没有cookie信息，即使加入上面的隐藏域代码，发现又可以访问了
+ 结论：django的csrf不是完全的安全
+ 当提交请求时，中间件'django.middleware.csrf.CsrfViewMiddleware'会对提交的cookie及隐藏域的内容进行验证，如果失败则返回403错误

### 验证码
+ 在用户注册、登录页面，为了防止暴力请求，可以加入验证码功能，如果验证码错误，则不需要继续处理，可以减轻一些服务器的压力
+ 使用验证码也是一种有效的防止crsf的方法
+ 验证码效果如下图：

![](http://p3ek8hcdl.bkt.clouddn.com/image/verifycode.png)

#### 验证码视图
+ 新建viewsUtil.py，定义函数verifycode
+ 此段代码用到了PIL中的Image、ImageDraw、ImageFont模块，需要先安装Pillow（3.4.1）包，详细文档参考http://pillow.readthedocs.io
+ Image表示画布对象
+ ImageDraw表示画笔对象
+ ImageFont表示字体对象，ubuntu的字体路径为“/usr/share/fonts/truetype/freefont”
+ 代码如下：
```
from django.http import HttpResponse
def verifycode(request):
    #引入绘图模块
    from PIL import Image, ImageDraw, ImageFont
    #引入随机函数模块
    import random
    #定义变量，用于画面的背景色、宽、高
    bgcolor = (random.randrange(20, 100), random.randrange(
        20, 100), 255)
    width = 100
    height = 25
    #创建画面对象
    im = Image.new('RGB', (width, height), bgcolor)
    #创建画笔对象
    draw = ImageDraw.Draw(im)
    #调用画笔的point()函数绘制噪点
    for i in range(0, 100):
        xy = (random.randrange(0, width), random.randrange(0, height))
        fill = (random.randrange(0, 255), 255, random.randrange(0, 255))
        draw.point(xy, fill=fill)
    #定义验证码的备选值
    str1 = 'ABCD123EFGHIJK456LMNOPQRS789TUVWXYZ0'
    #随机选取4个值作为验证码
    rand_str = ''
    for i in range(0, 4):
        rand_str += str1[random.randrange(0, len(str1))]
    #构造字体对象
    font = ImageFont.truetype('FreeMono.ttf', 23)
    #构造字体颜色
    fontcolor = (255, random.randrange(0, 255), random.randrange(0, 255))
    #绘制4个字
    draw.text((5, 2), rand_str[0], font=font, fill=fontcolor)
    draw.text((25, 2), rand_str[1], font=font, fill=fontcolor)
    draw.text((50, 2), rand_str[2], font=font, fill=fontcolor)
    draw.text((75, 2), rand_str[3], font=font, fill=fontcolor)
    #释放画笔
    del draw
    #存入session，用于做进一步验证
    request.session['verifycode'] = rand_str
    #内存文件操作
    import cStringIO
    buf = cStringIO.StringIO()
    #将图片保存在内存中，文件类型为png
    im.save(buf, 'png')
    #将内存中的图片数据返回给客户端，MIME类型为图片png
    return HttpResponse(buf.getvalue(), 'image/png')
    
    

python3中需要把import cStringIO换成io  StringIO换成BytesIO
#内存文件操作
    import io
    buf = io.BytesIO()
    #将图片保存在内存中，文件类型为png
    im.save(buf, 'png')
    #将内存中的图片数据返回给客户端，MIME类型为图片png
    return HttpResponse(buf.getvalue(), 'image/png')
```
#### 配置url
+ 在urls.py中定义请求验证码视图的url
```
from . import viewsUtil

urlpatterns = [
    url(r'^verifycode/$', viewsUtil.verifycode),
]
```
#### 显示验证码
+ 在模板中使用img标签，src指向验证码视图
```
<img id='verifycode' src="/verifycode/" alt="CheckCode"/>
```
+ 启动服务器，查看显示成功
+ 扩展：点击“看不清，换一个”时，可以换一个新的验证码
```
<script type="text/javascript" src="/static/jquery-1.12.4.min.js"></script>
<script type="text/javascript">
    $(function(){
        $('#verifycodeChange').css('cursor','pointer').click(function() {
            $('#verifycode').attr('src',$('#verifycode').attr('src')+1)
        });
    });
</script>
<img id='verifycode' src="/verifycode/?1" alt="CheckCode"/>
<span id='verifycodeChange'>看不清，换一个</span>
```
+ 为了能够实现提交功能，需要增加form和input标签
```
<form method='post' action='/verifycodeValid/'>
    <input type="text" name="vc">
    <img id='verifycode' src="/verifycode/?1" alt="CheckCode"/>
<span id='verifycodeChange'>看不清，换一个</span>
<br>
<input type="submit" value="提交">
</form>
```
#### 验证
+ 接收请求的信息，与session中的内容对比
```
from django.http import HttpResponse

def verifycodeValid(request):
    vc = request.POST['vc']
    if vc.upper() == request.session['verifycode']:
        return HttpResponse('ok')
    else:
        return HttpResponse('no')
```
+ 配置验证处理的url
```
urlpatterns = [
    url(r'^verifycodeValid/$', views.verifycodeValid),
]
```

## 高级
### 静态文件处理
+ 项目中的CSS、图片、js都是静态文件

#### 配置静态文件
+ 在settings 文件中定义静态内容
```
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
```
+ 在项目根目录下创建static目录，再创建当前应用名称的目录
```
mysite/static/myapp/
```
+ 在模板中可以使用硬编码
```
/static/my_app/myexample.jpg
```
+ 在模板中可以使用static模板标签
```
{ % load static from staticfiles %}
<img src="{ % static "my_app/myexample.jpg" %}" alt="My image"/>
会自动根据STATIC_URL生成url
```

### 中间件
+ 是一个轻量级、底层的插件系统，可以介入Django的请求和响应处理过程，修改Django的输入或输出
+ 激活：添加到Django配置文件中的MIDDLEWARE_CLASSES元组中
+ 每个中间件组件是一个独立的Python类，可以定义下面方法中的一个或多个
    + _init _：无需任何参数，服务器响应第一个请求的时候调用一次，用于确定是否启用当前中间件
    + process_request(request)：执行视图之前被调用，在每个请求上调用，返回None或HttpResponse对象
    + process_view(request, view_func, view_args, view_kwargs)：调用视图之前被调用，在每个请求上调用，返回None或HttpResponse对象
    + process_template_response(request, response)：在视图刚好执行完毕之后被调用，在每个请求上调用，返回实现了render方法的响应对象
    + process_response(request, response)：所有响应返回浏览器之前被调用，在每个请求上调用，返回HttpResponse对象
    + process_exception(request,response,exception)：当视图抛出异常时调用，在每个请求上调用，返回一个HttpResponse对象
+ 使用中间件，可以干扰整个处理过程，每次请求中都会执行中间件的这个方法
+ 示例：自定义异常处理
+ 与settings.py同级目录下创建myexception.py文件，定义类MyException，实现process_exception方法
```
from django.http import HttpResponse
class MyException():
    def process_exception(request,response, exception):
        return HttpResponse(exception.message)
```
+ 将类MyException注册到settings.py中间件中
```
MIDDLEWARE_CLASSES = (
    'test1.myexception.MyException',
    ...
)
```
+ 定义视图，并发生一个异常信息，则会运行自定义的异常处理

### 上传图片
+ 当Django在处理文件上传的时候，文件数据被保存在request.FILES
+ FILES中的每个键为<input type="file" name="" />中的name
+ 注意：FILES只有在请求的方法为POST 且提交的<form>带有enctype="multipart/form-data" 的情况下才会包含数据。否则，FILES 将为一个空的类似于字典的对象
+ 使用模型处理上传文件：将属性定义成models.ImageField类型
```
pic=models.ImageField(upload_to='cars/')
```
+ 注意：如果属性类型为ImageField需要安装包Pillow
```
pip install pillow
```
+ 图片存储路径
    + 在项目根目录下创建media文件夹
    + 图片上传后，会被保存到“/static/media/cars/图片文件”
    + 打开settings.py文件，增加media_root项
```
MEDIA_ROOT=os.path.join(BASE_DIR,"static/media")
```
+ 使用django后台管理，遇到ImageField类型的属性会出现一个file框，完成文件上传
+ 手动上传的模板代码
```
<html>
<head>
    <title>文件上传</title>
</head>
<body>
    <form method="post" action="upload/" enctype="multipart/form-data">
        <input type="text" name="title"><br>
        <input type="file" name="pic"/><br>
        <input type="submit" value="上传">
    </form>
</body>
</html>
```
+ 手动上传的视图代码
```
from django.conf import settings

def upload(request):
    if request.method == "POST":
        f1 = request.FILES['pic']
        fname = '%s/cars/%s' % (settings.MEDIA_ROOT,f1.name)
        # python3中需‘wb’模式打开
        with open(fname, 'w') as pic:
            for c in f1.chunks():
                pic.write(c)
        return HttpResponse("ok")
    else:
        return HttpResponse("error")
        
```

### Admin站点
+ 通过使用startproject创建的项目模版中，默认Admin被启用
+ 1.创建管理员的用户名和密码
```
python manage.py createsuperuser
然后按提示填写用户名、邮箱、密码
```
+ 2.在应用内admin.py文件完成注册，就可以在后台管理中维护模型的数据
```
from django.contrib import admin
from models import *

admin.site.register(HeroInfo)
```
+ 查找admin文件：在INSTALLED_APPS项中加入django.contrib.admin，Django就会自动搜索每个应用的admin模块并将其导入

#### ModelAdmin对象
+ ModelAdmin类是模型在Admin界面中的表示形式
+ 定义：定义一个类，继承于admin.ModelAdmin，注册模型时使用这个类
```
class HeroAdmin(admin.ModelAdmin):
    ...
```
+ 通常定义在应用的admin.py文件里
+ 使用方式一：注册参数
```
admin.site.register(HeroInfo,HeroAdmin)
```
+ 使用方式二：注册装饰器
```
@admin.register(HeroInfo)
class HeroAdmin(admin.ModelAdmin):
```
+ 通过重写admin.ModelAdmin的属性规定显示效果，属性主要分为列表页、增加修改页两部分

#### 列表页选项
##### “操作选项”的位置
+ actions_on_top、actions_on_bottom：默认显示在页面的顶部
```
class HeroAdmin(admin.ModelAdmin):
    actions_on_top = True
    actions_on_bottom = True
```
##### list_display
+ 出现列表中显示的字段
+ 列表类型
+ 在列表中，可以是字段名称，也可以是方法名称，但是方法名称默认不能排序
+ 在方法中可以使用format_html()输出html内容
```
在models.py文件中
from django.db import models
from tinymce.models import HTMLField
from django.utils.html import format_html

class HeroInfo(models.Model):
    hname = models.CharField(max_length=10)
    hcontent = HTMLField()
    isDelete = models.BooleanField()
    def hContent(self):
        return format_html(self.hcontent)

在admin.py文件中
class HeroAdmin(admin.ModelAdmin):
    list_display = ['hname', 'hContent']
```
+ 让方法排序，为方法指定admin_order_field属性
```
在models.py中HeroInfo类的代码改为如下：
    def hContent(self):
        return format_html(self.hcontent)
    hContent.admin_order_field = 'hname'
```
+ 标题栏名称：将字段封装成方法，为方法设置short_description属性
```
在models.py中为HeroInfo类增加方法hName：
    def hName(self):
        return self.hname
    hName.short_description = '姓名'
    hContent.short_description = '内容'

在admin.py页中注册
class HeroAdmin(admin.ModelAdmin):
    list_display = ['hName', 'hContent']
```
##### list_filter
+ 右侧栏过滤器，对哪些属性的值进行过滤
+ 列表类型
+ 只能接收字段
```
class HeroAdmin(admin.ModelAdmin):
    ...
    list_filter = ['hname', 'hcontent']
```
##### list_per_page
+ 每页中显示多少项，默认设置为100
```
class HeroAdmin(admin.ModelAdmin):
    ...
    list_per_page = 10
```
##### search_fields
+ 搜索框
+ 列表类型，表示在这些字段上进行搜索
+ 只能接收字段
```
class HeroAdmin(admin.ModelAdmin):
    ...
    search_fields = ['hname']
```

#### 增加与修改页选项
+ fields：显示字段的顺序，如果使用元组表示显示到一行上
```
class HeroAdmin(admin.ModelAdmin):
    ...
    fields = [('hname', 'hcontent')]
```
+ fieldsets：分组显示
```
class HeroAdmin(admin.ModelAdmin):
    ...
    fieldsets = (
        ('base', {'fields': ('hname',)}),
        ('other', {'fields': ('hcontent',)}),
    )
```
+ fields与fieldsets两者选一

#### InlineModelAdmin对象
+ 类型InlineModelAdmin：表示在模型的添加或修改页面嵌入关联模型的添加或修改
+ 子类TabularInline：以表格的形式嵌入
+ 子类StackedInline：以块的形式嵌入
```
class HeroInline(admin.TabularInline):
    model = HeroInfo

class BookAdmin(admin.ModelAdmin):
    inlines = [
        HeroInline,
    ]
```
#### 重写admin模板
+ 在项目所在目录中创建templates目录，再创建一个admin目录
+ 设置模板查找目录：修改settings.py的TEMPLATES项，加载模板时会在DIRS列表指定的目录中搜索
```
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```
+ 从Django安装的目录下（django/contrib/admin/templates）将模板页面的源文件admin/base_site.html拷贝到第一步建好的目录里
+ 编辑base_site.html文件
+ 刷新页面，发现以刚才编辑的页面效果显示
+ 其它管理后台的模板可以按照相同的方式进行修改

### 分页
+ Django提供了一些类实现管理数据分页，这些类位于django/core/paginator.py中

#### Paginator对象
+ Paginator(列表,int)：返回分页对象，参数为列表数据，每页数据的条数

##### 属性
+ count：对象总数
+ num_pages：页面总数
+ page_range：页码列表，从1开始，例如[1, 2, 3, 4]

##### 方法
+ page(num)：下标以1开始，如果提供的页码不存在，抛出InvalidPage异常

##### 异常exception
+ InvalidPage：当向page()传入一个无效的页码时抛出
+ PageNotAnInteger：当向page()传入一个不是整数的值时抛出
+ EmptyPage：当向page()提供一个有效值，但是那个页面上没有任何对象时抛出

#### Page对象
##### 创建对象
+ Paginator对象的page()方法返回Page对象，不需要手动构造

##### 属性
+ object_list：当前页上所有对象的列表
+ number：当前页的序号，从1开始
+ paginator：当前page对象相关的Paginator对象

##### 方法
+ has_next()：如果有下一页返回True
+ has_previous()：如果有上一页返回True
+ has_other_pages()：如果有上一页或下一页返回True
+ next_page_number()：返回下一页的页码，如果下一页不存在，抛出InvalidPage异常
+ previous_page_number()：返回上一页的页码，如果上一页不存在，抛出InvalidPage异常
+ len()：返回当前页面对象的个数
+ 迭代页面对象：访问当前页面中的每个对象

#### 示例
+ 创建视图pagTest
```
def pagTest(request, pindex):
    list = HeroInfo.objects.all()
    p = Paginator(list, 4)
    if pindex == '':
        pindex = '1'
    pindex = int(pindex)
    page = p.page(pindex)
    plist = p.page_range
    context = {
        'page':page,
        'plist':plist,
        'pindex':pindex
    }
    return render(request, 'booktest/pagtest.html', context)
```
+ 配置url
```
url(r'^pagTest(\d*)/$', views.pagTest),
```
+ 定义模板pagTest.html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    {%for hero in page%}
    <li>{{hero.hname}}</li>
    {%endfor%}
</ul>
{%if page.has_previous%} <!-- 有上一页的话才显示‘上一页’ -->
<a href="/pagTest{{page.previous_page_number}}/">上一页</a>
{%endif%}
{%for p in plist%} <!-- 循环页码列表 -->
{%if pindex == p%} <!-- 当前页面不给链接 -->
<a>第{{p}}页</a>
{%else%}
<a href="/pagTest{{p}}/">第{{p}}页</a>
{%endif%}
{%endfor%}
{%if page.has_next%} <!-- 有下一页的话才显示‘下一页’ -->
<a href="/pagTest{{page.next_page_number}}/">下一页</a>
{%endif%}
</body>
</html>
```
![](http://p3ek8hcdl.bkt.clouddn.com/image/page.gif)

### ajax
#### 使用Ajax
+ 使用视图通过上下文向模板中传递数据，需要先加载完成模板的静态页面，再执行模型代码，生成最终的html，返回给浏览器，这个过程将页面与数据集成到了一起，扩展性差
+ 改进方案：通过ajax的方式获取数据，通过dom操作将数据呈现到界面上
+ 推荐使用框架的ajax相关方法，不要使用XMLHttpRequest对象，因为操作麻烦且不容易查错
+ jquery框架中提供了$.ajax、$.get、$.post方法，用于进行异步交互
+ 由于csrf的约束，推荐使用$.get
+ 示例：实现省市区的选择
+ 最终实现效果如图：
![](http://p3ek8hcdl.bkt.clouddn.com/image/city.gif)

##### 引入js文件
+ js文件属于静态文件，创建目录结构如图：
![](http://p3ek8hcdl.bkt.clouddn.com/image/js.jpg)

##### 修改settings.py关于静态文件的设置
```
STATIC_URL = '/static/'
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, 'static'),
]
```
##### 在models.py中定义模型
```
class AreaInfo(models.Model):
    atitle = models.CharField(max_length=20)
    aparent = models.ForeignKey('self', null=True, blank=True)
```
##### 生成迁移
```
python manage.py makemigrations
python manage.py migrate
```
##### 向表中填充示例数据
+ [测试数据](http://p3ek8hcdl.bkt.clouddn.com/txt/area.txt)
+ 注意将表的名称完成替换

##### 在views.py中编写视图
+ index用于展示页面
+ getArea1用于返回省级数据
+ getArea2用于根据省、市编号返回市、区信息，格式都为字典对象
```
from django.shortcuts import render
from django.http import JsonResponse
from models import AreaInfo

def areaTest(request):
    return render(request, 'booktest/areaTest.html')

def getArea1(request):
    list = AreaInfo.objects.filter(aparent__isnull=True)
    list2 = []
    for a in list:
        list2.append([a.id,a.atitle])
    return JsonResponse({'data':list2})

def getArea2(request, pid):
    list = AreaInfo.objects.filter(aparent=pid)
    list2 = []
    for a in list:
        list2.append([a.id,a.atitle])
    return JsonResponse({'data':list2})
```
##### 在urls.py中配置urlconf
```
url(r'^areaTest/$', views.areaTest),
url(r'^getArea/$', views.getArea1),
url(r'^(\d+)/$', views.getArea2),
```
##### 定义模板areaTest.html
+ 修改settings.py的TEMPLATES项，设置DIRS值
```
'DIRS': [os.path.join(BASE_DIR, 'templates')],
```
+ 定义模板文件：包含三个select标签，分别存放省市区的信息
```
<!DOCTYPE html>
<html>
<head>
    <title>省市区列表</title>
</head>
<body>
<select id="pro">
    <option value="">请选择省</option>
</select>
<select id="city">
    <option value="">请选择市</option>
</select>
<select id="dis">
    <option value="">请选择区县</option>
</select>
</body>
</html>
```
##### 在模板中引入jquery文件
```
<script src="/static/jquery-1.12.4.min.js"></script>
```
##### 编写js代码
+ 绑定change事件
+ 发出异步请求
+ 使用dom添加元素
```
<script>
        $(function(){
            //页面加载完获取所有省
            $.get('/getArea/',function (data) {
                var pro = $('#pro');
                $.each(data.data,function (index,item) {
                    pro.append('<option value="'+ item[0] +'">'+ item[1] +'</option>');
                });
            });
            
            //省下拉列表改变时获取下面的市
            $('#pro').change(function () {

                $.get('/'+$(this).val()+'/',function (data) {
                    //每次先清空再添加 防止一直追加
                    var city = $('#city');
                    city.empty().append('<option value="">请选择市</option>');
                    var dis=$('#dis');
                    dis.empty().append('<option value="">请选择区县</option>');
                    $.each(data.data,function (index,item) {
                        city.append('<option value="'+ item[0] +'">'+ item[1] +'</option>');
                    });
                })
            });

            //市下拉列表改变时获取下面的区县
            $('#city').change(function () {
                $.get('/'+$(this).val()+'/',function (data) {
                    var dis = $('#dis');
                    dis.empty().append('<option value="">请选择区县</option>');
                    $.each(data.data,function (index,item) {
                        dis.append('<option value="'+ item[0] +'">'+ item[1] +'</option>');
                    });
                })
            });
            
        });
    </script>
```
