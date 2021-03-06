---
title: Ubuntu|搭建 matrix 服务
date: 2022-04-18 11:35:24
updated: 
tags: 
- matrix
- synapse
---

记录一些搭建 matrix 的经历

<!--more-->

## 配置Postgresql

安装 `PostgreSQL`

```shell
sudo apt install postgresql
```

进入 `postgres` 用户

```shell
sudo su - postgres
```

创建用户和数据库

```postgresql
# this will prompt for a password for the new user
createuser --pwprompt synapse_user

createdb --encoding=UTF8 --locale=C --template=template0 --owner=synapse_user synapse
```

## 修改homeserver.yaml

这部分参照官方说明来改就行了

```shell
sudo nano /etc/matrix-synapse/homeserver.yaml
```

## 安装TLS证书

`snap package` 是安装 `certbot` 最容易的方式

```shell
sudo apt install snapd
sudo snap install core
```

检查安装正常

```shell
sudo snap install hello-world
hello-world
```

安装 `certbot`

```shell
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

打开 `port 80`，`443`

```shell
ufw allow 80
ufw allow 443
```

>  尤其注意打开 `port 80`，容易忘记

获取证书。

```shell
sudo certbot certonly --nginx -d synapse.matrix.org
```

接下来按照提示，输入邮箱等信息，成功获取证书会有信息提示。

> Saving debug log to /var/log/letsencrypt/letsencrypt.log
> Requesting a certificate for synapse.matrix.org
>
> Successfully received certificate.
> Certificate is saved at: /etc/letsencrypt/live/synapse.matrix.org/fullchain.pem
> Key is saved at:         /etc/letsencrypt/live/synapse.matrix.org/privkey.pem
> This certificate expires on 2022-07-15.
> These files will be updated when the certificate renews.
> Certbot has set up a scheduled task to automatically renew this certificate in the background.
>
> If you like Certbot, please consider supporting our work by:
>  * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
>  * Donating to EFF:                    https://eff.org/donate-le

## 配置NGINX

安装 `NGINX`

```shell
sudo apt-get install nginx -y
```

新建并打开 `NGINX` 配置

```shell
sudo nano /etc/nginx/sites-available/synapse.matrix.org.conf
```

在 `synapse.matrix.org.conf` 填入配置，配置参考官方说明和别人的教程。

载入新配置

```shell
sudo ln -s /etc/nginx/sites-available/synapse.matrix.org.conf /etc/nginx/sites-enabled/
```

重启 `NGINX`

在[官方的测试网站](https://federationtester.matrix.org/)，测试 `synapse` 是否正常运行

如果连接没有问题，会显示

> \> Checks        Success

以及

> DNS results
> server name/.well-known result contains explicit port number: no SRV lookup done

## 注册新用户

开始注册新用户

```shell
register_new_matrix_user -c /etc/matrix-synapse/homeserver.yaml http://localhost:8008
```

> homeserver.yaml 的地址要写出来，不然会报错

按照提示为新用户增加信息

```
New user localpart: erikj
Password:
Confirm password:
Make admin [no]:   # no / yes
Success!
```

### 禁止用户自己注册新用户

修改 homeserver.yaml

```
enable_registration: false
```

### 手动重置用户密码

参考 [MATRIX - reset password (synapse)](https://blog.dtpnk.tech/en/matrix-reset-password-synapse/)，`SQLite`  和 `postgreSQL` 有各自的操作

我的数据库是 `postgreSQL`，首先获得新密码的哈希值

```shell
hash_password
```

进入  `postgreSQL`

```shell
su postgres
psql
```

依次输入一些数据库操作

```postgresql
\connect synapse
 UPDATE users SET 
 password_hash='$2a$12$q...sp3m' WHERE name='@username:synapse.matrix.org';
\q
```

> * `\connect synaps`e 是连接到 synapse 的数据库，这里的 `synapse` 是 你在 `homeserver.yaml` 写的数据库名字
> * `password_hash` 为上面计算出的新密码哈希值
> * `WHERE name` 后面要换成你自己的用户名

## 配置 turn 语音服务

根据官方教程，就可以配置了，但是官方教程没有指出

```shell
sudo nano /etc/default/coturn
```

去除 `TURNSERVER_ENABLED=1` 的注释

## 添加管理面板 synapse-admin

```dockerfile
docker run -d -p 8090:80 awesometechnologies/synapse-admin # 8090 是 synapse-admin 出现的端口号
```

> 在设置 reverse-proxy 的时候，需要 access to the following endpoints:
>
> - `/_matrix`
>
> - `/_synapse/admin`

如果设置没问题，在 `synapse-admin` 页面输入 `Homeserver URL` 的时候，应该显示 synpase 服务器的版本号

## 附录

### 生成需要长度的密钥

```shell
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1
```

### 修改hba_file

官网上关于 `postgresSQL` 需要修改 `hba_file` 的内容，[有说明](https://github.com/matrix-org/synapse/blob/develop/docs/postgres.md)。如果提示 `FATAL: Ident authentication failed for user "synapse_user"`，需要更换除了`ident` 以外的数据库认证方式。

这里说明一下如何找到 `hba_file` 的位置，进入 `psql` （需要先进入 `postgres 用户`）

>  进入和退出 psql，使用 \q;

向 `postgresSQL` 询问 `hba_file` 的位置

```postgresql
SHOW hba_file；
```

#### 关于 Hostwinds 更换IP

通过[工具](https://www.vps234.com/ipchecker/)检测 ip 情况，国内叉国外勾，说明 ip 可能需要换一个了。

如果是 [Hostwinds](https://www.hostwinds.com/) ，可以免费更换 ip 。这时候在中间灰色一栏选择 `Manage IP’s` ，再点击最右边的 `Fix ISP Block` 进行一个换 ip 的操作，耐心等待一会再ping。

> 注意：
>
> - 选择Fix ISP Block，不要选成Change Main IP了，后面那个换 ip 要加钱
> - account 信息中 country 必须填写 China，才有这个免费服务

### 删除package相关

查询目前存在的 package 名称和地址

```postgresql
dpkg-statoverride --list 
```

移除 package

```postgresql
dpkg-statoverride --remove <path>
```

### 可能用到的 NGINX 相关命令

```shell
sudo nginx -t
sudo nginx -s reload     # 开启 nginx 之后，修改了配置，只需要用这个命令
sudo systemctl enable nginx
sudo systemctl restart nginx
sudo systemctl status nginx
```

### 可能用到的  synapse 相关命令

```shell
sudo systemctl enable matrix-synapse
sudo systemctl restart matrix-synapse
sudo systemctl status matrix-synapse
```

### 移除文件的命令

注意：rm 是永久移除

```shell
sudo rm /etc/nginx/sites-enabled/synapse.matrix.org.conf 
```

### 退出 > 的方法

输入`\q;`

> 一定要有分号

### 配置和使用 ufw

记得要打开port 22，[教程](https://linuxize.com/post/how-to-setup-a-firewall-with-ufw-on-ubuntu-18-04/)

### Nano操作

- CTRL键 + 6 （或按住 转移 并移动光标）以标记设置并标记您想要的内容。
- ALT + 6 用于复制标记的文本。
- 用Ctrl+Y到上一页，Ctrl+V到下一页

### scp命令

用 [scp](https://superuser.com/questions/414803/how-to-scp-from-linux-server-to-windows-client) 可以实现从当前用户拷贝文件到别的用户。

## 参考

感谢 `狐狸` 和 `chn`，对我的网络之旅的帮助。

###### 官方教程

[1] https://github.com/matrix-org/synapse/blob/develop/docs/setup/installation.md

[2] https://github.com/matrix-org/synapse/blob/develop/docs/postgres.md

[3] https://github.com/matrix-org/synapse/blob/develop/docs/reverse_proxy.md

###### 其它

[1] https://blog.southfox.me/2022/04/%E6%90%AD%E5%BB%BAMatrix%E5%8D%B3%E6%97%B6%E9%80%9A%E4%BF%A1%E6%9C%8D%E5%8A%A1/

[2] https://tech.minnix.dev/projects/build-your-own-matrix-server-behind-your-existing-nginx-reverse-proxy

[3] https://www.informaticar.net/install-matrix-synapse-on-ubuntu-20-04/

[4] https://www.atlantic.net/vps-hosting/how-to-install-matrix-synapse-on-ubuntu-20-04/
