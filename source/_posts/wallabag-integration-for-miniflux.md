---
title: 安装 wallabag 作为 miniflux 的插件
date: 2022-05-16 13:47:23
tags:
- miniflux
- wallabag
- rss
---

`wallabag ` 有一些抓取不全的问题，用作 `miniflux` 的插件就还好。本文说明了如何使用 docker-compose 安装 `wallabag` ，以及在 `miniflux` 中使用 `wallabag`。

<!--more-->

## 安装 wallabag (docker-compose)

`docker-compose.yml` 的填写参考[官方说明](https://github.com/wallabag/docker#docker-compose)，各个环境变量的说明在[这里](https://github.com/wallabag/docker#environment-variables)

```
version: '3'
services:
  wallabag:
    image: wallabag/wallabag  
    container_name: wallabag  # 自定义容器名
    depends_on:    
      - db
      - redis
    ports:
      - 自定义端口号:80  # 前面的是为自定义端口，80不可变
    environment:
      - MYSQL_ROOT_PASSWORD=wallaroot
      - SYMFONY__ENV__DATABASE_DRIVER=pdo_mysql
      - SYMFONY__ENV__DATABASE_HOST=db
      - SYMFONY__ENV__DATABASE_PORT=3306
      - SYMFONY__ENV__DATABASE_NAME=wallabag
      - SYMFONY__ENV__DATABASE_USER=wallabag
      - SYMFONY__ENV__DATABASE_PASSWORD=wallapass
      - SYMFONY__ENV__DATABASE_CHARSET=utf8mb4
      # 建议直接用自己的ip地址:端口号，否则用可能显示界面不全
      - SYMFONY__ENV__DOMAIN_NAME=http://自定义你的域名:端口号 
      # 邮箱设置(下面是默认设置的值，其实不设置也不影响使用)
      - SYMFONY__ENV__MAILER_HOST=127.0.0.1
      - SYMFONY__ENV__MAILER_USER=~
      - SYMFONY__ENV__MAILER_PASSWORD=~
      - SYMFONY__ENV__FROM_EMAIL=wallabag@example.com
    volumes: # 冒号之前的目录名是上文要求你新建的文件夹地址
      - /wallabag/images:/var/www/wallabag/web/assets/images
      - /wallabag/data:/var/www/wallabag/data
    restart: always

  db:
    image: mariadb
    container_name: wallabag_mariadb
    environment:
      - MYSQL_ROOT_PASSWORD=wallaroot
    volumes: # 冒号之前的目录名是上文要求你新建的文件夹地址
      - /wallabag/mysql:/var/lib/mysql
    restart: always

  redis:
    image: redis:alpine
    container_name: wallabag_redis
    restart: always
```

然后是自己用 NGINX 等设置 reverse-proxy

输入网址，即可管理 `wallabag` 和创建新用户

> 创建完成的 `wallabag` 实例，有默认的管理帐号： 用户名和密码都是 wallabag （记得登录上去修改密码）
>
> 可以直接在 web 端，使用管理帐号创建新帐号，或者 enable 新注册的帐号

## 在 miniflux 中使用 wallabag integration

integration 的添加引用参考 `miniflux` 的 [Integration with External Services](https://miniflux.app/docs/services.html#wallabag)，就可以完成了

然后就可以在 `miniflux` ，save 文章到 `wallabag` 稍后读啦~

wallabag 的 `re-fetch content`，帮我弥补了忘记在 miniflux 设置全文抓取了问题。但是，我现在发现 wallabag 的抓取问题有：

* 对于 podcast，抓取不了附带的音频，但是 miniflux 能做到。

* 对于有些代码块，显示奇怪，比如，行号和内容会分开，但是 miniflux 不会出问题

* 对于有些文章，内容本来就只有几行，会显示 reload 失败，其实是已经抓取了全文了
