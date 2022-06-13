---
title: Docker-compose|安装和使用 miniflux
date: 2022-04-24 20:04:27
updated:
tags:
- miniflux
- RSShub
- RSS
- docker-compose
---

> 安装部分参考 [Docker 快速搭建 Miniflux + RSSHub ](https://www.jkg.tw/p3246/)，讲解细致，全程复制粘贴就可以完成！所以不需要我多写啦。

这里主要想记录一下使用 `miniflux + RSSHub` 进行 `RSS订阅` 的笔记~

## docker-compose.yml

创建目录

```shell
mkdir ~/miniflux && cd ~/miniflux
nano docker-compose.yml
```

写入 `docker-compose.yml`

```dockerfile
version: "3"

services:

   miniflux:
     image: miniflux/miniflux:latest
     container_name: miniflux
     restart: unless-stopped
     ports:
       - "8888:8080"
     depends_on:
       - db
       - rsshub
     environment:
       - DATABASE_URL=postgres://miniflux:somepass888@db/miniflux?sslmode=disable
       - POLLING_FREQUENCY=15
       - RUN_MIGRATIONS=1

   db:
     image: postgres:latest
     container_name: postgres
     restart: unless-stopped
     environment:
       - POSTGRES_USER=miniflux
       - POSTGRES_PASSWORD=somepass888
     volumes:
       - miniflux-db:/var/lib/postgresql/data

   rsshub:
     image: diygod/rsshub:latest
     container_name: rsshub
     restart: unless-stopped
     ports:
       - "1200:1200"
     environment:
       NODE_ENV: production
       CACHE_TYPE: redis
       REDIS_URL: "redis://redis:6379/"
       PUPPETEER_WS_ENDPOINT: "ws://browserless:3000"
     depends_on:
       - redis
       - browserless

   browserless:
     image: browserless/chrome:latest
     container_name: browserless
     restart: unless-stopped

   redis:
     image: redis:alpine
     container_name: redis
     restart: unless-stopped
     volumes:
       - redis-data:/data

volumes:
  miniflux-db:
  redis-data:
```

启动服务

```shell
docker-compose up -d
```

在 `miniflux` 目录下，继续初始化 `miniflux`

```shell
# miniflux 大版本升级也可能用到这一条
docker-compose exec miniflux /usr/bin/miniflux -migrate
```

> 输出显示：
>
> -> Current schema version: 56
> -> Latest schema version: 56

新增 miniflux 管理帐号，用于登录管理后台

```shell
docker-compose exec miniflux /usr/bin/miniflux -create-admin
```

> * 根据输出的提示输入用户名与密码
>
> * 如果忘记了 miniflux 管理员帐号密码，就这个命令重新建立一个管理员帐号，去网页里管理

之后用 nginx / apache / caddy 等，设置 reverse proxy

## 一般内容的 RSS 订阅

1. 首先，在 miniflux 选择 Feeds > Add subscription 

2. 然后在 [RSSHub 官网](https://docs.rsshub.app/) 查找**路由**

   > 打开官网需要科学

3. 根据前面提到的教程的介绍，在**路由**前面加上 `http://rsshub:1200` ，得到 URL

   - 如果路由为 `/bilibili/user/followers/:uid`
   - 那么获得的 URL 格式为：`http://rsshub:1200`/bilibili/user/followers/:uid

4. 把得到的 URL 填入 miniflux 的订阅界面

## miniflux 抓取全文

在 miniflux 选择 Feeds > 一个订阅的空间 > Edit > `Fetch original content` (需要页面拉到最下面)

## 其余（Newletter）订阅

这部分由于还没开始用，所以就没写。用到哪里，写到哪里~
