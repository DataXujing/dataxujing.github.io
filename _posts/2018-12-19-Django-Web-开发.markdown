---
layout: post
title: "Django Web开发"
img: bowen63.png
date: 2018-12-19 12:00:00 
description: You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
tag: [Python,Django,pyecharts]
---

#### 徐静 2018年12月技术培训交流

### 1. Django模式

Django遵循MVC模式：MVC中重要的思想就是解耦合，各自干自己的事情。

+ M:Model,模型，与数据库进行交互
+ V:View,视图 html页面
+ C:Controller，控制器，接收请求，处理，返回数据，与视图进行交互

比如一个登陆网页(视图),点击登录时，将账号和密码发送给MVC框架中的控制器(Controller),我们从控制器中进行出炉，需要查询数据库，但这里不直接操作数据库，通过模型层(Model)去操作数据库，而在Django中把控制器变成了模板(Template),叫做MVT。

![]({{site.url}}/assets/bowen63/django.jpg)


### 2.创建一个项目

+ 1.创建一个虚拟环境 virtualenv my_project_env
+ 2.虚拟环境中安装必要的模块，我们这里用到了：
  - pip3 install django # 最新版的
  - pip3 install pyecharts
  - pip3 install numpy
  - pip3 install pandas
  - pip3 install request
  - pip3 install bs4
  - pip3 install virtualenv
  - pip3 install uwsgi
  - pip3 install supervisor
+ 3.开启一个项目
  - django-admin startproject qdFang

```python
qdFang:

settings.py: 这个是整个项目的配置文件
urls.py:     路由的总控制文件
wsgi.py:     和web服务(如apache,wsgi)配合使用的配置文件

manage.py   django项目的管理文件,很多命令号的操作在这里

```

```python
setting:

DATABASES: 数据库的设置，默认sqlite3数据库没有用户名和密码
STATIC_URL:静态资源文件路径
TEMPLATES: 模板页面文件路径,html
MIDDLEWARE_CLASSES:中间件，添加功能
INSTALLED_APPS: 项目的模块
ALLOWED_HOSTS: 访问服务的IP地址

时区及语言
	LANGUAGE_CODE = 'zh-Hans'
	TIME_ZONE = 'Asia/Shanghai'

```

+ 4.一个项目是由多个APP组成的
  - python manage.py startapp fangAna

```python

fangAna:

admin.py:把数据库注册到这个文件后，可以在admin界面下使用
models.py:数据库文件，ORM映射关系，数据库创建语句
views.py:具体功能文件，是一个又一个的函数组成
urls.py:APP下的路由控制文件

```
开启一个APP之后，要记得把他在setting.py下写入到INSTALLED_APPS中

+ 5.运行一个测试服务器
  - python manage.py runserver ip:port

### 3.ORM映射

Django中内置ORM框架。运用这个框架让我们在操作数据库的时候更简单。
不用再写一些简单的Sql语句，通过创建一个类，来一一对应数据库中表的字段。
通过操作类来操作数据库中的表。所以在Models中就写和数据库中表，字段一样的类。
同时，他也可以根据你创建的类，来给你创建数据库中的表。

### 4.Models

```python

# 类名字就是表名 ，继承models.Model表示是一个模型
class Lijia2(models.Model):
    # 姓名 CharField(max_length=20) 表示varchar(20)
    stuname = models.CharField(max_length=20)
    # 生日 DateField日期类型
    bri_date = models.DateField()
```

id主键不用我们写，他会自动生成

其他类型：

BooleanField(default=False) # bool类型，default默认值

ForeignKey('表名') # 设置外键 生成表的时候格式是  字段名_id ,赋值需要直接给对象。
如果我想让他生成一个表：

分为两步：先生成迁移文件，利用迁移文件再生成表

1: python manage.py makemigrations 根据你对数据库的修改，策略迁移文件

2: python manage.py migrate  生成对应的SQL语句

通过模型层去操作数据库！

### 5.admin(后台管理)

1、修改成中文页面

修改setting文件，找到LANGUAGE_CODE='en-us'

把他修改成：LANGUAGE_CODE='zh-hans'

2、中国时间

找到TIME_ZONE = 'UTC'修改成

TIME_ZONE = 'Asia/Shanghai'

3、创建管理员账户

执行命令：python manage.py createsuperuser

会提示输入用户名，邮箱（可以随意写）和密码。自己设置一个就好了

4、运行项目

运行命令：python manage.py runserver

5、登陆管理员页面

在浏览器中输入127.0.0.1:8000/admin ，就会进入管理员页面，登陆就好了。

6、注册模型类

在admin.py中注册模型类，来帮助我们生成对应的管理页面。

对我们的lijia2表进入注册。

在admin.py中添加：

from .models import Lijia2
admin.site.register(Lijia2)

```python
我们看到显示的是一个英文，我想显示我的名字怎么办？
这个stuinfo object 其实是我们str(Lijia2)将一个对象转化为字符串的结果，
所以我们只需要在Models中的stuinfo中重写__str__方法即可。
def __str__(self):
    return  self.xiaoqu

自定义管理页面
在admin.py中创建自定义管理的类

class Lijia2Admin(admin.ModelAdmin):
	list_display = ['id','xiaoqu','mianji',]

admin.site.register(Lijia2.Lijia2Admin)

```

### 6.views

```python
# 导入模块
from django.http import HttpResponse
from django.shortcuts import render

定义index方法，必须要有参数，（用于接收参数）
def index(request):
    return  HttpResponse('我是index页')

def home(request):
	data_dict = {}
    return  render(request,'我是index页',data_dict)

```

### 7.templates 和 static

+ 具体的在setting.py中的修改见项目
+ 模板类似于flask中的Jinja2

有了以上这些不一定能做出一个项目，需要自己去尝试。


### 8.其他

最后以一个实际部署生产环境的真实django项目为例，演示django web应用的效果

+ 为什么在这里我们使用Django而非Flask？
+ django文档及学习资源丰富
+ django的高级应用[django数据迁移，多数据库连用，用户注册及表单，缓存系统，中间件，微信，支付宝接口，单元测试]
+ 展示一个青岛2手房的Web应用
	- 演示说明项目构建过程
	- 如何在生产环境中部署django
	- 后台定时爬虫
	- pyecharts结合django的使用