---
title: 安装和设置 Apache，NGINX，MySQL，PHP
date: 2022-03-31 11:26:20
update: 2022-04-01 21:51:56
tags: 
- VPS
- Apache
- NGINX
- MySQL
- PHP
- reverse
---

## Apache 和 NGINX 对比

根据[NGINX vs Apache – Choosing the Best Web Server in 2022](https://www.hostinger.com/tutorials/nginx-vs-apache-what-to-use/)，Apache 和 NGINX的优缺点主要为：

- Apache 一次处理一个连接请求，NGINX 可以同时处理多个连接请求。
- NGINX 提供静态内容更快，但是需要另外的软件帮助处理动态内容，而 Apache 可以自己提供动态内容。
- Apache 提供 **.htaccess** 文件，可以在不更改主服务器设置，就能改变网站的设置。

> 性能最好的方法是：把 **NGINX** 作为反向代理，放在 **Apache** 前面。

所以，我将先后安装 **Apache** 和 **NGINX**。

## 安装和配置 Apache

> 在安装NGINX之前，需要先安装并配置好Apache。

很可能你的 VPS 已经安装了 **Apache**，输入下列命令可以检查是否存在 **Apache**。

```
sudo systemctl status apache2
```

显示 VPS 没有安装 **Apache**。

![image-20220331204125554](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220331204125554.png)

自己安装 **Apache**：

```
sudo apt install apache2
```

![image-20220331204333918](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220331204333918.png)

如果 VPS 之前有防火墙，需要为 **Apache** 新建立一条 rule：

```
sudo ufw allow “Apache Full”
```

再次检查 **Apache** 是否存在：

```
sudo systemctl status apache2
```

![image-20220331205813262](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220331205813262.png)

> 输入 q 离开上面的页面

最后，打开一个自己电脑上的浏览器，输入 VPS 的 IP 地址，如果出现下面的页面，就是 **Apache** 运行好了。

![image-20220331205630378](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220331205630378.png)

## 安装 MySQL

```
sudo apt-get install mysql-server
```

> 安装过程中，会要求设置 **root**  user 的密码，不要让密码为空白。
>
> 如果没有提示设置 **root** 密码，可以输入
>
> ```
> mysql_secure_installation
> ```
>
> 进行初始密码的设定

检查 **MySQL** 服务的状态：

```
sudo systemctl status mysql
```

如果 **MySQL** 工作正常，会输出：

![image-20220331211011132](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220331211011132.png)

> 输入 q 离开上面的页面

## 安装PHP

> 可能需要安装 **nano** 文本编辑器： 
>
> sudo apt-get install nano

检查 packet 更新：

```
sudo apt-get upgrade
```

安装 **PHP**：

```
sudo apt-get install php
```

测试 **PHP** 运行状态：

​	简单的方法是，输入：**php -v**

​	复杂一点的方法是：

> 在 **/var/www/html** 目录下，新建一个 **test.php**，并用 **nano** 打开。这个目录被称为 **webroot**，是 **Apache** 查找网页的缺省位置（如果没有被设置从哪里找的话）。
>
> ```
> sudo nano /var/www/html/test.php
> ```
>
> 在新建的文件中，输入之后，按 **CTRL + X** ，**Y** 保存后 **ENTER** 离开。
>
> ```
> <?php
> phpinfo();
> ?>
> ```
>
> 打开浏览器，输入网址来测试 **PHP**
>
> ```
> http://<your_vps_ip_adress>/test.php
> ```
> 移除测试文件（一定要移除，否则有安全风险）
>
> ```
> sudo rm /var/www/html/test.php
> ```

## Ubuntu release 版本升级

就是突然想升个级

```
sudo apt update
sudo apt upgrade
sudo do-release-upgrade
```

中间某部分输出：

![image-20220401111409746](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220401111409746.png)

参考官网文档：[Upgrading|Ubuntu](https://ubuntu.com/server/docs/upgrade-introduction)

更新好了，允许 **reboot** 后重新连接，显示：

![image-20220401112506423](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220401112506423.png)

## 安装和配置 NGINX

### 安装 **NGINX**

```
sudo apt-get update
sudo apt-get install nginx
```
### 为 NGINX 配置 Apache

sudo nano（用nano，不要用vim，我觉得vim太难用了，可以的话还要记得删除vim的包）

```
sudo nano /etc/apache2/ports.conf
```

在第 5 行，把端口号 80 改为 8080.

```
Listen 8080
```

> 8000 和 8080 都是 HTTP 协议的备用端口号

在 **/etc/apache2/sites-available/** 目录下，创建文件 **000-default.conf**

```
sudo nano /etc/apache2/sites-available/000-default.conf
```

确保在 **000-default.conf** 文件中，配置如下：（把端口改为 **8080**，改为 8000 也可以 ）

```
<VirtualHost *:8080>
    ServerName www.reverse.com
    ServerAlias reverse.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

设置完成后，按照上面 PHP 安装时候，检查是否正确的方法，查看浏览器网页：

> <ip 地址>：8080/info.php

### 为 Apache 配置 NGINX

关闭 default 的虚拟主机（visual host）

```
sudo unlink /etc/nginx/sites-enabled/default
```

打开：

```
sudo nano /etc/nginx/nginx.conf
```

![Enable gzip and proxying.](https://www.howtoforge.com/images/how-to-install-nginx-as-reverse-proxy-for-apache-on-ubuntu-15-10/2.png?ezimgfmt=rs:550x199/rscb5/ng:webp/ngcb5)

创建新文件：

```
sudo nano /etc/nginx/sites-available/reverse.conf
```

复制下列配置：

> 注意：You could try the following two commands to paste from the clipboard. Both of them should work.
>
> 1. **Ctrl+Shift+v**
> 2. **Shift+Insert**
>
> The **Ctrl+U** command only allows pasting text that was copied or cut from within nano itself, hence the reason the command is not working.

```
server {
    listen 80 default_server;
    index index.php index.html index.htm;
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

保存并离开 nano，然后开启虚拟主机配置：

```
sudo ln -s /etc/nginx/sites-available/reverse.conf /etc/nginx/sites-enabled/
```

测试 NGINX

```
sudo nginx -t
```

![image-20220401214639290](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220401214639290.png)

> 也可以用这个命令：
>
> sudo service nginx configtest
>
> ![image-20220401215005520](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220401215005520.png)

重启 NGINX：

```
sudo systemctl restart nginx
```

现在，在浏览器中输入 VPS 的 IP 地址，会被反向代理到 Apache 的页面。

------

## 参考资料

[1] [What Is Apache? An In-Depth Overview of Apache Web Server](https://www.hostinger.com/tutorials/what-is-apache)

[2] [How to Install Laravel on Ubuntu 18.04 with Apache](https://www.hostinger.com/tutorials/how-to-install-laravel-on-ubuntu-18-04-with-apache-and-php/#1_Install_Apache_Web_Server)

[3] [How to Install Linux, Apache, MySQL, PHP (LAMP) stack on Ubuntu 16.04](https://www.hostinger.com/tutorials/how-to-install-lamp-ubuntu-16-04#How_LAMP_Works)

[4] [How to Install PHP on Any Server (Linux, macOS, Windows)](https://kinsta.com/blog/install-php/#php-prerequisites)

[5] [How to set up and login as root user in MySQL](https://www.fosslinux.com/47280/login-as-root-user-in-mysql.htm)

[6] [Upgrading|Ubuntu](https://ubuntu.com/server/docs/upgrade-introduction)

[7] [How to Set Up an Nginx Reverse Proxy](https://www.hostinger.com/tutorials/how-to-set-up-nginx-reverse-proxy/#1_Install_Nginx)

[8] [How to Set Up Nginx as a Reverse Proxy For Apache on Debian 11](https://linuxhostsupport.com/blog/how-to-set-up-nginx-as-a-reverse-proxy-for-apache-on-debian-11/)

[9] [How to Use Nginx to Redirect](https://www.hostinger.com/tutorials/nginx-redirect/)



