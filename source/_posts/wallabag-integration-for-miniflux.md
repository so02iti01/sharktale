---
title: Docker-compose|安装 wallabag 作为 miniflux 的插件
date: 2022-05-16 13:47:23
updated:
tags:
- Miniflux
- Wallabag
- RSS
- docker-compose
---

`wallabag ` 的优点是可以下载并保存网页，防止链接失效无法访问收藏的内容。`wallabag ` 有一些抓取不全的问题，用作 `miniflux` 的插件就还好。本文说明了如何使用 docker-compose 安装 `wallabag` ，以及在 `miniflux` 中使用 `wallabag`。

<!--more-->

## 安装 wallabag (docker-compose)

`docker-compose.yml` 的填写参考[官方说明](https://github.com/wallabag/docker#docker-compose)，各个环境变量的说明在[这里](https://github.com/wallabag/docker#environment-variables)

```dockerfile
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
      # SYMFONY__ENV__DOMAIN_NAME不能用 127.0.0.1:端口号，也不能用 ip地址:端口号
      # 否则将出现加入reverse proxy之后，wallabag图标主题等显示不全的问题
      - SYMFONY__ENV__DOMAIN_NAME=https://wallabag.example.com  # 修改为自己的域名
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

然后是自己用 caddy 设置了 reverse-proxy

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

## caddy 的简要使用方法
在设置 `caddy` 的时候遇到了一些问题，于是再次重新设置了，记一下（万一以后又出错了）

caddy 可以自动获取 Let’s Encrypt SSL certificate 并自动续期，设置也比 NGINX 简单。 

> caddy 的设置参考了[Docker 快速搭建 Miniflux + RSSHub](https://www.jkg.tw/p3246/)

下载 `caddy`
```bash
# 下载编译好的 Caddy 执行档
wget https://github.com/caddyserver/caddy/releases/download/v2.0.0-beta.15/caddy2_beta15_linux_amd64
# 赋予执行和设定低端口绑定的权限
chmod +x caddy2_beta15_linux_amd64 && sudo setcap cap_net_bind_service=+ep caddy2_beta15_linux_amd64
# 把执行档放在系统文件夹并改名
sudo mv caddy2_beta15_linux_amd64 /usr/local/bin/caddy
```

创建 `caddyfile`
```bash
sudo mkdir /etc/caddy && sudo nano /etc/caddy/Caddyfile
```

写入 `caddyfile`
```caddyfile
wallabag.example.com {
        encode zstd gzip
        reverse_proxy localhost:8090  # 修改为自己的端口号
}
```

防火墙打开 port 80 和 port 443，为 `caddy` 申请 SSL certificate 做准备
```bash
sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
sudo apt install iptables-persistent  # 将上面2个防火墙规则自动永久载入本机
```
试运行 caddyfile
```bash
sudo caddy run --config /etc/caddy/Caddyfile
```
> 如果没问题，会看见显示一连串的 INFO，包括获取证书的提示。此时访问设置好的域名，将可以访问服务。确认没问题之后，`ctrl + C` 中断 caddyfile 执行

下面要做的是设置系统每次重启，都会自动启动 caddy

新建设置文件
```bash
sudo nano /etc/systemd/system/caddy.service
```
写入
```bash
[Unit]
Description=Caddy Server
After=syslog.target
After=network.target
[Service]
User=root
Group=root
LimitNOFILE=64000
ExecStart=/usr/local/bin/caddy run --config /etc/caddy/Caddyfile
Restart=always
[Install]
WantedBy=multi-user.target
```

启用 caddy
```bash
sudo systemctl daemon-reload
sudo systemctl enable caddy.service
sudo systemctl start caddy.service
```

## 管理 MariaDB

进入 `MariaDB`

```mariadb
mysql -u root -p -h localhost
```

显示全部 databases

```mariadb
SHOW DATABASES;
```

使用某一个 database

```mariadb
USE <databasename>;
```

展示选择的 database 的 tables

```mariadb
SHOW tables;
```

显示 table 里面的数据

```mariadb
SHOW [FULL] {COLUMNS | FIELDS} FROM tbl_name [FROM db_name]
    [LIKE 'pattern' | WHERE expr];
```

* 例如：

```mariadb
SHOW COLUMNS FROM mytable FROM mydb;
SHOW COLUMNS FROM mydb.mytable;
```

