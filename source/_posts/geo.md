title: "geo"
date: 2015-07-14 13:45:47
tags:
---

Windows GeoDjango PostgreSql Postgis
[参考](https://docs.djangoproject.com/en/dev/ref/contrib/gis/install/#windows) 

都选了32位的版本
PostgreSQL 在安装过程注意选择 Locale [Chinese Marco S.R.C]，不然安装失败

*OSGeo4W* 安装后使用管理员身份执行 geodjango_setup.bat

安装完成后 `python manage.py migrate` 怎么都不行
也用管理员身份执行 居然可以了，但是配置的用户必须得是superuser
