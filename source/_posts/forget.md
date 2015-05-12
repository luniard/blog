title: "forget"
date: 2015-05-12 15:06:30
tags:
- 忘记
- Django

categories: 日志

---

说着要写写博客，折腾着折腾着就忘记了，甚至忘了这玩意怎么用了
趁着打算研究下 Python + Django, 顺便记下环境的搭建

#搭建

下载 [setuptools-7.0](https://pypi.python.org/packages/source/s/setuptools/setuptools-7.0.tar.gz) 解压
``` bash
cd setuptools-7.0
python setup.py install
```

下载 [Django-1.8.1](https://www.djangoproject.com/m/releases/1.8/Django-1.8.1.tar.gz)
``` bash
cd Django-1.8.1
python setup.py install
```
下载 [postgreSql扩展支持](file:///F:/download/psycopg2-2.6.0.win32-py2.7-pg9.4.1-release.exe)
直接安装

配置环境变量将 *C:\Python27\Scripts* 加入到 *Path*


`$ django-admin startproject mysite`
Database setup:
提前建好库
``` bash
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'MYSITE',
        'USER': 'xxx',
        'PASSWORD': 'xxx',
        'HOST': '172.16.xx.xx',
        'PORT': '5432'
    }
}
```

`$ python manage.py migrate`
查看数据库表的变化

`$ python manage.py runserver`

checkout with http://127.0.0.1:8000/

#Creating models¶
上文建立的是一个project， 可以在一个project上建立多个app
`$ python manage.py startapp polls`

编辑polls/models.py
```
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')


class Choice(models.Model):
    question = models.ForeignKey(Question)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```


编辑mysite/settings.py
```
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'polls',
)
```

`$ python manage.py makemigrations polls`
`$ python manage.py sqlmigrate polls 0001`