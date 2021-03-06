---
title: Docker-compose|Nextcloud 安装与可用 container commands
date: 2022-05-05 11:09:03
updated:
tags: 
- Nextcloud
- docker-compose
---

使用 `docker-compose` 安装 [Nextcloud](https://nextcloud.com/install/#instructions-server)，`Caddy` 作为 reverse-proxy 以及获取 SSL 证书

<!--more-->

## docker-compose 安装 Nextcloud

这里参照的官方的 `Base version - apache` version，使用 apache image 和 mariaDB

> 这个方法自己是没有 SSL ，后面我采用 Caddy 实现 SLL 和 reverse-proxy

由于之前我的 vps 上已经安装过了 docker-compose 和 Caddy，所以直接从写 `docker-compose.yml` 开始

```shell
# 新建目录和docker-compose.yml文件
sudo mkdir ~/nextcloud && cd ~/nextcloud
sudo nano docker-compose.yml
```

在 `docker-compose.yml` 中粘贴一下内容，并保存

```dockerfile
version: '2'

volumes:
  nextcloud:
  db:

services:
  db:
    image: mariadb:10.5
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password1    # 这里要填写自己的密码,要与后面2个密码不一样
      - MYSQL_PASSWORD=password2   # 这里要填写自己的密码
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud

  app:
    image: nextcloud
    restart: always
    ports:
      - 8080:80     # 如果之前已经占用了8080端口，记得换成别的端口
    links:
      - db
    volumes:
      - nextcloud:/var/www/html
    environment:
      - MYSQL_PASSWORD=password2      # 这里要填写自己的密码
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db

```

运行 `docker-compose up -d` 上线容器后，在浏览器输入 `服务器IP地址:8080/` 可以看到你的 Nextcloud，继续用图形界面注册 `管理员帐号` 以及 `完成安装`

> 如果在 docker-compose.yml 中选择了其它端口号，也要在这里换成别的端口号

## 设置 Caddy

由于这台机器上已经安装了 Caddy（在之前安装 miniflux 的时候），所以这里直接继续使用 Caddy

```shell
 # 打开上次写过的 Caddyfile
 sudo nano /etc/caddy/Caddyfile
```

假如你把 Nextcloud 放在 `site.example.com`  ，在 Caddyfile 末尾增加上以下内容

```caddyfile
https://site.example.com {
    encode zstd gzip
    reverse_proxy 127.0.0.1:8080    # 这个端口号与 docker-compose.yml 中端口号保持一致
}
```

> 如果在同一个 vps 上运行多个网站，需要为这些网站都 添加 DNS 的 A 记录

启用新的 Caddy Service，Caddy 就会自动为你申请 SSL 证书啦

```shell
sudo systemctl daemon-reload   
sudo systemctl enable caddy.service
sudo systemctl start caddy.service
```

然后你就可以输入网址，开始你的 Nextcloud 之旅啦

## Caddy 使用遇到的问题与解决

1. 一开始直接  `reload caddy`  之后，显示

> run: loading initial config: loading new config: starting caddy administration endpoint: listen tcp 127.0.0.1:2019: bind: address already in use

这是由于已有进程使用 port `2019` ，看到[有人通过改 port 为 2020 得到了解决](https://caddy.community/t/tcp-127-0-0-1-bind-address-already-in-use/10661/3)（我用的上一章末的三行代码解决），在整个 Caddyfile 最前面增加

```
{
    admin 0.0.0.0:2020
}
```

如果还是没有得到解决的话，根据对于 [admin](https://caddyserver.com/docs/caddyfile/options#admin) 的说明，可以设置关闭 `admin endpoint`。但是这样做的缺点是，caddy 的配置改变后需要停止再重启服务。

```caddyfile
{
    admin off
}
```

2. 遇到了 443 端口被占用的问题

> run: loading initial config: loading new config: http app module: start: tcp: listening on :443: listen tcp :443: bind: address already in use

这是由于之前已经运行了一个带有 SSL 的网站导致的，参考 [Issue](https://github.com/caddyserver/caddy/issues/309)，直接 `caddy stop`，然后

```shell
sudo systemctl daemon-reload   # 这一行特别重要
sudo systemctl enable caddy.service
sudo systemctl start caddy.service
```
## Nextcloud Security
Nextcloud 提供了 [Nextcloud Security Scan](https://scan.nextcloud.com/)，可以从外部对服务器进行检查，根据检查的结果反馈进行安全提升。

如果显示 `__Host-Prefix` 有问题，参考 [Overwrite parameters](https://docs.nextcloud.com/server/16/admin_manual/configuration_server/reverse_proxy_configuration.html#overwrite-parameters)，在 `config/config.php` 中添加这下面这行
```
'overwriteprotocol' => 'https',
```
> 这里提到的 `config/config.php` ，是 Nextcloud 存储 config 的地方，可以采用 `find /var -name "config.php"` 查找

还要在 Nextcloud 的用户界面内，点击右上角的头像，选择 Setting > Overview 进行安全检查，会出现很多警告，按照提示进行进一步更改。

## 一些问题的解决

遇到了问题 `Access through untrusted domain`

> Please contact your administrator. If you are an administrator, edit the "trusted_domains" setting in config/config.php like the example in config.sample.php.
>
> Further information how to configure this can be found in the [documentation](https://docs.nextcloud.com/server/24/go.php?to=admin-trusted-domains).

查找 config.php 位置，进而修改

```shell
find /var -name "config.php"
```

`server1.example.com` 是服务所在域名，修改为

```php
'trusted_domains' =>
  array (
   0 => 'localhost',
   1 => 'server1.example.com',    
   2 => '192.168.1.50',
   3 => '[fe80::1:50]',
),
```

或者使用环境变量，Start the container with:

```
-e NEXTCLOUD_TRUSTED_DOMAINS=cloud.yourdomain.tld
```

## Nextcloud (docker)的一些可用命令

```bash
docker exec -it -u www-data nextcloud_app_1 php occ # 查询 nextcloud 所有可用命令，这里的 nextcloud_app_1 是我的 Container_Name
```

查询 `Container_Name` 可用

```shell
docker ps
```

如果需要迁移data，或许会需要去除 server-side 的 加密，并为所有文件解除加密

```shell
docker exec -it -u www-data nextcloud_app_1 php occ encryption:decrypt-all  # 这里的 nextcloud_app_1 是我的 Container_Name
```

> Disable server side encryption... done.
>
> You are about to start to decrypt all files stored in your Nextcloud.
> It will depend on the encryption module and your setup if this is possible.
> Depending on the number and size of your files this can take some time
> Please make sure that no user access his files during this process!



