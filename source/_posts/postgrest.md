---
title: 使用postgrest全自动生成面向表的RESTful接口
date: 2018-10-10 15:58:38
categories: 程序人生
tags:
    - PostgreSQL
    - RESTful
---

尝试使用了一下`postgrest`，用法非常简单，但是功能非常强大。不仅能生成相应的`RESTful`接口，更是连`swagger`文档都给准备好了，简直意外惊喜。
关于使用，如果有`docker`经验的话，直接看这个`compose`文件就好了
```
# postgrest.yml

version: '3'
services:
  server:
    image: postgrest/postgrest
    ports:
      - "3000:3000"
    links:
      - db:db
    environment:
      PGRST_DB_URI: postgres://app_user:password@db:5432/app_db
      PGRST_DB_SCHEMA: public
      PGRST_DB_ANON_ROLE: app_user #In production this role should not be the same as the one used for the connection
    depends_on:
      - db
  db:
    image: postgres
    ports:
      - "5433:5432"
    environment:
      POSTGRES_DB: app_db
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: password
  # Uncomment this if you want to persist the data.
  # volumes:
  #   - "./pgdata:/var/lib/postgresql/data"
  swagger:
    image: swaggerapi/swagger-ui
    ports:
      - "8080:8080"
    expose:
      - "8080"
    environment:
      API_URL: http://127.0.0.1:3000/
```
运行命令
```
docker-compose -f postgrest.yml up
```
就可以顺利启动好了。浏览器访问`http://127.0.0.1:8080`，就能看到`swagger`的界面，而`RESTful`接口则运行在`http://127.0.0.1:3000`。
当然此时因为没有创建表，所以还没有接口可供使用。只要去数据库创建几张表，再回来刷新`swagger`页面，就能看到全套的`RESTful`接口喽。