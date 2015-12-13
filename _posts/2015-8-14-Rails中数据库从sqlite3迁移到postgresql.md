---
date: 2015-8-14 20:53:12+00:00
layout: post
title: Rails中数据库从sqlite3迁移到postgresql
thread: 22
categories: Rails开发笔
tags: Rails PostgreSql
---

最近在一步一步的按照[railstutorial](https://www.railstutorial.org/)学习RoR的知识。Rails默认使用的sqlite3，数据库的切换中遇到了很多问题，过程中也参考了很多的文档，[heroku](heroku.com)官方的数据库迁移指南，以及[stackoverflow](http://stackoverflow.com/)上面的各种回答。

其中最好的方式肯定是在new一个项目的时候制定是用postgresql。

`rails new myapp --database=postgresql`

这样就避免了修改PROJ/config/database.yml文件。

但是现实总是不幸的，我们在创建的时候可能不知道后面需要使用postgresql，就像我一样当时使用了系统默认的，当想push到heroku上的时候发现数据库要迁移，而且官网也建议本地测试的数据库和heroku的数据库保持一致。

数据库从sqlite3迁移到postgresql你需要做一下的事情，首先肯定是本地安装postgresql，各种系统的安装方法可以参考官方的文档。下面看下和rails相关的部分。

.在本地创建用户和密码

`sudo -u postgres createuser -s my_app`

登录到指定的用户

`sudo -u postgres my_app`

为这个用户设置密码

`postgres=#` `\password secret`

修改本地的/config/database.yml文件
```
``` 

    default: &default
    adapter: postgresql
    encoding: unicode
    pool: 5
    timeout: 5000
    host: localhost
    username: my_app
    passsword: secret
    development:
         <<: *default
        database: sample_app_development
    test:
        <<: *default
       database: sample_app_test
    production:
       <<: *default
     database: sample_app_production

然后执行`rake db:create:all`就会创建相应的数据库。中间可能会提示：

>PG::InsufficientPrivilege: ERROR:  permission denied to create database

>: CREATE DATABASE "sample_app_development" ENCODING = 'unicode'

这里是由于我们的用户没有建表的权限，你可以通过一下的方式给他加上相应的权限。
`sudo -u postgres my_app`登录到相应的用户。然后

>postgres=# `alter ROLE my_app CREATEDB;`

>ALTER ROLE

>postgres=# `\q`

-END


