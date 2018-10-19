# vueAndDjango
我们首先使用Django来搭建web后端api框架。

1、 先在终端敲入命令：

django-admin startproject myproject

目录结构：


2、 进入项目根目录，创建一个app：

python manage.py startapp myapp

目录结构：


3、 在myproject下的settings.py配置文件中，把默认的sqllite3数据库换成我们的mysql数据库：

# Database
# https://docs.djangoproject.com/en/1.11/ref/settings/#databases

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'myproject',
        'USER': 'root',
        'PASSWORD': 'root',
        'HOST': '127.0.0.1',
    }
}
并把app加入到installed_apps列表里：

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
     'myapp',
]
4、 在app目录下的models.py里我们简单写一个model如下：

# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.db import models

# Create your models here.
class Book(models.Model):
    book_name = models.CharField(max_length=64)
    add_time = models.DateTimeField(auto_now_add=True)

    def __unicode__(self):
        return self.book_name
只有两个字段，书名book_name和添加时间add_time。如果没有指定主键的话django会自动新增一个自增id作为主键

5、 在app目录下的views里我们新增两个接口，一个是show_books返回所有的书籍列表（通过JsonResponse返回能被前端识别的json格式数据），二是add_book接受一个get请求，往数据库里添加一条book数据：

# Create your views here.
@require_http_methods(["GET"])
def add_book(request):
    response = {}
    try:
        book = Book(book_name=request.GET.get('book_name'))
        book.save()
        response['msg'] = 'success'
        response['error_num'] = 0
    except  Exception,e:
        response['msg'] = str(e)
        response['error_num'] = 1

    return JsonResponse(response)

@require_http_methods(["GET"])
def show_books(request):
    response = {}
    try:
        books = Book.objects.filter()
        response['list']  = json.loads(serializers.serialize("json", books))
        response['msg'] = 'success'
        response['error_num'] = 0
    except  Exception,e:
        response['msg'] = str(e)
        response['error_num'] = 1

    return JsonResponse(response)
可以看出，在ORM的帮忙下，我们的接口实际上不需要自己去组织SQL代码

6、 在app目录下，新增一个urls.py文件，把我们新增的两个接口添加到路由里：

from django.conf.urls import url, include
import views

urlpatterns = [
url(r'add_book$', views.add_book, ),
url(r'show_books$', views.show_books, ),
]

我们还要把app下的urls添加到project下的urls中，才能完成路由：
from django.conf.urls import url, include
from django.contrib import admin
from django.views.generic import TemplateView
import myapp.urls

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^api/', include(myapp.urls)),
    url(r'^$', TemplateView.as_view(template_name="index.html")),
]
在项目的根目录，输入命令：
python manage.py makemigrations myapp

python manage.py migrate

查询数据库，看到book表已经自动创建了：


在项目的根目录，输入命令：
python manage.py runserver

启动服务，通过postman测试一下我们刚才写的两个接口：

add_book


show_books


四、 构建Vue.js前端项目
1、 先用npm安装vue-cli脚手架工具（vue-cli是官方脚手架工具，能迅速帮你搭建起vue项目的框架）：

    `npm install -g vue-cli`
安装好后，在project项目根目录下，新建一个前端工程目录：

    vue-init webpack appfront  //安装中把vue-router选上，我们须要它来做前端路由
进入appfront目录，运行命令：

    npm install //安装vue所须要的node依赖
现在我们可以看到整个文件目录结构是这样的：


2、 在目录src下包含入口文件main.js，入口组件App.vue等。后缀为vue的文件是Vue.js框架定义的单文件组件，其中标签中的内容可以理解为是类html的页面结构内容，标签中的是js的方法、数据方面的内容，而则是css样式方面的内容：


3、 我们在src/component文件夹下新建一个名为Library.vue的组件，通过调用之前在Django上写好的api，实现添加书籍和展示书籍信息的功能。在样式组件上我们使用了饿了么团队推出的element-ui，这是一套专门匹配Vue.js框架的功能样式组件。由于组件的 编码涉及到了很多js、html、css的知识，并不是本文的重点，因此在此只贴出部分代码：


4、 在src/router目录的index.js中，我们把新建的Home组件，配置到vue-router路由中：


5、 如果发现列表抓取不到数据，可能是出现了跨域问题，打开浏览器console确认：


这时候我们须要在Django层注入header，用Django的第三方包django-cors-headers来解决跨域问题：

       pip install django-cors-headers
settings.py 修改：

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

CORS_ORIGIN_ALLOW_ALL = True
  注意中间件的添加顺序
6、 在前端工程目录下，输入npm run dev启动node自带的服务器，浏览器会自动打开， 我们能看到页面：


尝试新增书籍，新增的书籍信息会实时反映到页面的列表中，这得益于Vue.js的数据双向绑定特性。

在前端工程目录下，输入npm run build，如果项目没有错误的话，就能够看到所有的组件、css、图片等都被webpack自动打包到dist目录下了：

五、 整合Django和Vue.js
目前我们已经分别完成了Django后端和Vue.js前端工程的创建和编写，但实际上它们是运行在各自的服务器上，和我们的要求是不一致的。因此我们须要把Django的TemplateView指向我们刚才生成的前端dist文件即可。

1、 找到project目录的urls.py，使用通用视图创建最简单的模板控制器，访问 『/』时直接返回 index.html:

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^api/', include(myapp.urls)),
    url(r'^$', TemplateView.as_view(template_name="index.html")),
]
2、 上一步使用了Django的模板系统，所以需要配置一下模板使Django知道从哪里找到index.html。在project目录的settings.py下：

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': ['appfront/dist'],
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
3、 我们还需要配置一下静态文件的搜索路径。同样是project目录的settings.py下：

# Add for vuejs
STATICFILES_DIRS = [
    os.path.join(BASE_DIR, "appfront/dist/static"),
]
4、 配置完成，我们在project目录下输入命令python manage.py runserver，就能够看到我们的前端页面在浏览器上展现：


注意服务的端口已经是Django服务的8000而不是node服务的8080了
