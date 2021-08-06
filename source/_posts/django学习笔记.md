---
title: django学习
date: 2021-04-04 10:58:30
tags: 编程
---
# django学习



#### Web基本概念——前置

##### socket 套接字

位于应用层与传输层之间的虚拟层,通过此接口进行数据传输等操作

##### c/s 架构

即client/server架构,通过客户端程序与服务端通信

##### b/s 架构

即browser/server架构,通过浏览器发送http请求,获得服务器的资源

便捷、通用

##### 请求过程

###### 0.socket服务端 绑定/监听

​	0.0 导入socket库

```python
import socket
```

​	0.1 创建socket服务端 	

```python
server = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
# socket.AF_INET 表示使用ipv4
# socket.SOCK_STREAM 表示使用tcp
```

​	0.2 绑定到本地端口

```python
socket.bind(('127.0.0.1',8080))
# target与port要以元组的形势传入
```

​	0.3 开启监听

```python
socket.listen(5)
# 设置最大连接数
```

​	0.4 等待连接

```python
while True:
	so,addr = socket.accept()
  # 返回连接socket与客户端地址
```

###### 1.socket客户端 连接

​	0.1 创建socket客户端

```python
client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
```

​	0.2 连接服务端

```python
client.connect((target,port))
```

###### 2.双方传输数据

```python
data = server.recv(1024)
client.send(b'')
```

###### 3.处理请求、发送内容

###### 4.重复到 2 

###### 5.断开连接

##### HTTP

###### Hyper Text Transfer Protocol 超文本传输协议

###### 推荐书籍:《http权威指南》

###### Web中作为请求与响应的标准

请求: 请求行 请求头请求数据

![](https://images2018.cnblogs.com/blog/867021/201803/867021-20180322001733298-201433635.jpg)

```http
GET / HTTP/1.1
Host: 127.0.0.1:8800
Connection: keep-alive
sec-ch-ua: "Google Chrome";v="89", "Chromium";v="89", ";Not A Brand";v="99"
sec-ch-ua-mobile: ?0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 11_2_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: hblid=LXa5cZvOpVbu5wDe3m39N0HIgrB8BFbo; csrftoken=NQ2jvGjSCUXBQsAU0TgOCoop939FrKCBhyPS7jAgHBlTEtpqgLOvOjZqirQ0gjeQ; security_level=0
\r\n
```

响应: 状态行 响应头部 响应数据

![](https://images2018.cnblogs.com/blog/867021/201803/867021-20180322001744323-654009411.jpg)

###### http默认端口80,https默认端口443

###### 响应码

1xx消息 请求已被服务器接收,继续处理

2xx成功 请求已成功被服务器接收、理解并接受

​	200 成功

3xx重定向 需要后续操作才能完成此请求

4xx请求错误 客户端请求含有错误或无法执行

​	403 客户端访问权限不足

​	404 客户端请求资源不存在

5xx服务器错误 服务器在处理正确请求时发生错误

###### 请求方法

GET、HEAD、POST、PUT、DELETE、TRACE、OPTIONS、CONNECT

###### URL

统一资源定位符,用于从互联网获取信息

*Protocol://Domain:Port/Path?Key1=Value1#Part*

#### django——正文

##### 下载

`pip3 install django -i https://pypi.tsinghua.edu.cn/simple`

##### 创建项目

`django-admin startproject PROJECT_NAME`

[^注意]: 需要在settings中更改DIRS路径(Template)

或在professional pycharm中创建Django项目

[^注意]: pycharm创建django3项目时,需要在settings.py中加上import os

##### 项目目录

#####  |── djangoStudy  项目文件夹

#####  │    |── __init__.py
│    |── asgi.py
│    |── settings.py 配置
│    |── urls.py 路由 -> URL和函数的对应关系
 |     |── wsgi.py runserver命令使用wsgiref模块当作简单的webserver
 |── manage.py
 |── templates

##### 启动项目

`python3 manage.py runserver [[ip:]port]`

Pycharm执行manage.py

##### 编写页面

```python
from django.shortcuts import HttpResponse, render
def index(request):

    # return HttpResponse('index') # 返回字符串
    return render(request, 'index.html')  # 返回页面
urlpatterns = [
    path('admin/', admin.site.urls),
    path('index/', index),
]
```

##### 逻辑处理

###### request类

*METHOD* 请求方法

​	*GET&POST*

*GET字典&POST字典*

​	`POST.get('Key') `获得提交的参数

###### redirect类

```python
from django.shortcuts import redirect
def login(request):
  if True:
    return redirect(PATH)
```

重定向

##### 创建app

###### 生成app

`django-admin startapp APP_NAME`

.
 |── __init__.py
 |── admin.py 管理后台
 |── apps.py app
 |── migrations
 |    |── __init__.py
 |── models.py 数据库
 |── tests.py
 |── views.py 函数

###### 安装(注册)app

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # 'app1',
    'app1.apps.App1Config',  # 推荐
]
```

##### app使用

###### views.py

将函数放在此文件内

同时更改urls.py

```python
from app1 import views
urlpatterns = [
    path('admin/', admin.site.urls),
    path('login/', views.login),
    path('login2/', views),
    path('index/', views.index)
]
```

##### models.py

###### Orm

对象关系映射

将对象与一张表一一对应,便捷

操作有限、牺牲执行效率

###### models

1.在settings.py中设置数据库驱动

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

2.在models.py中定义orm的关系

类 -> 表

对象 -> 数据行(记录)

属性 -> 字段

```python
class User(models.Model):  # 定义一张表，继承Model类
    username = models.CharField(max_length=32)  # varchar(32)
    password = models.CharField(max_length=32)
```

3.`python3 manage.py makemigrations` 检测所有app有什么变化,制作迁移文件

4.`python3 manage.py migrate  ` 根据迁移文件,将变更迁移到数据库

##### 数据库查询

```python
from app1 import models
# models.User.objects.get(username=user, password=pwd)
# 当get函数查询无结果或有多个结果时会报错
# 建议使用filter
models.User.objects.all().orderby('-id')#按照id字段逆序排列
if models.User.objects.filter(username=user, password=pwd):
  pass
```

##### 数据库增加

```python
from app1 import models
models.User.objects.create(name=name)
```

##### 数据库删除

```python
from app1 import models
models.User.objects.get(id=id).delete()
```

##### 数据库修改

```python
p = models.User.objects.filter(id=request.POST.get('id'))[0]
p.name = 'xxx'
p.save()
```



##### 修改数据库为mysql

1.创建mysql数据库

2.修改settings.py



```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'login',
        'HOST': 'localhost',
        'PORT': 3306,
        'USER': 'xxx',
        'PASSWORD': 'xxx',
    }
}
import pymysql
pymysql.install_as_MySQLdb()

```

3.写model

4.执行数据库迁移

`python3 manage.py makemigrations`

`Python3 manage.py migrate`

##### 模糊语法

```html
<table border="1">
        <thead>
        <tr>
            <th>序号</th>
            <th>id</th>
            <th>name</th>
        </tr>
        </thead>
        <tbody>
        {% for i in p_list %}
        <tr>
            <td>{{ forloop.counter }}</td>
            <td>{{ i.id }}</td>
            <td>{{ i.name }}</td>
        </tr>
        {% endfor %}
        </tbody>
    </table>
```



```python
return render(request, 'lists.html', context={'p_list': lists})
```



#### Django相关的命令 

##### 1.下载安装

`Pip3 install diango`创建 diango项目

`django-admin startproject`项目名称 启动项目

###### 切换到项目的根目录

`python3 manage. py runserver [127.0.0.1[:8000]]`运行服务

###### 创建app

`python3 manage. py startapp app`    名称 数据库迁移的命令

`python manage. py makemigrations` 检测app下的 model.py的变化记录下变更记录 

`python manage. py migrate` 迁移将变更记录同步到数据库中

##### 2.settings配置

​	BASE_DIR 项目的根目录

​	INSTALLED_APPS 注册的app

​	MIDDLEWARE 中间件

​		注释CSRF保护,可以提交POST请求

​	TEMPLATES模板

​		DIRS: [os.path.join(BASE_DIR, 'templates')] 

​	静态文件

​		STATIC_URL = '/static/' 静态文件的别名

​		STATICFILES_DIRS = [

BASE_DIR, 'static')				

​				]

##### 	3.django使用mysql

###### 		1.创建mysql数据库

###### 		2.配置settings

​			

```python
DATABASES
ENGINE django.db.backends.mysql
NAME 数据库名
HOST IP地址
PORT 3306
USER 用户名
PASSWORD 密码
```

###### 3.使用pymysql模块连接mysql数据库

建议写在\_\_init\_\_.py中

```python
import pymysql
pymysql.install_as_MySQLdb()
```

###### 4.在app的models.py写入model(models.model)

```
from django import models
class User(models.Model):
	username = models.CharField(max_length=32)
```

###### 5.迁移至数据库

```
python3 manage.py makemigrations
python3 manage.py migrate
```

##### 4.urls.py 路径与函数对应关系

```python
from app1 import views
urlpatterns = [
    path('admin/', admin.site.urls),
    path('list/', views.p_list),
    path('create/', views.create),
    path('delete/', views.delete),
    path('edit/', views.edit)
]
```

##### 5.函数

```python
def xxx(request):
	request.method  # 请求方式
	request.POST  # post传输的参数
  return
	# render(request, 'login.html', context={'info': 'Success'})  渲染并返回HTML页面
  # HttpResponse('字符串')
  # redirect('/index') 
```

##### 6.ORM

对象关系映射

类 -> 表

对象 -> 数据行

属性 -> 字段

###### 查询

```python
from app1 import models
models.User.objects.all()
models.User.object.get(id=1)
models.User.object.filter(id=1,name='x')
```

###### 增加

```python
models.User.objects.create(name='123')
```

###### 删除

```python
models.User.objects.get(id=1).delete()
models.User.objects.filter(id=1).delete()
```

###### 修改

```python
obj = models.User.objects.get(id=1)
obj.name = 'xxx'
obj.save()
```

##### 7.模版语法

```
return render(request, 'filename.html', context={'k':'1'})
返回并渲染一个html页面
```

```
{{  k1  }}  # 替换内容

{%  for i in K  %}
	{{  forloop.counter. }}
	{{ i.name  }}
{%  endfor  %}
```

