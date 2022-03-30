---
title: 新VPS需要的一些设置
date: 2022-03-29 15:18:29
tags: VPS
---

主要参考：[5 Steps to Get Your New Virtual Private Server (VPS) Ready to Use](https://www.hostinger.com/tutorials/getting-started-with-vps-hosting)
给自己记个笔记......万一那一天需要重新做呢。
<!-- more -->
##  安装 PuTTY 
使用 windows 电脑连接 remote 的 Linux VPS，需要一个图形化的界面。（当然也可以安装一个 Linux 的系统来操作）。想减少我短期内需要做的事，所以暂时选择前一种方法。

>  If your Windows is Windows 10 Version 1803, OpenSSH Client has been implemented as a Windows feature, so it's possible to use ssh command on command prompt without Putty and other SSH software.

![img](https://www.server-world.info/en/Ubuntu_18.04/ssh/img/13.png)

openSSH客户端的使用大同小异。下载 [PuTTY](https://www.putty.org/)，本次需要用到的是 **PuTTY** 和 **PuTTYgen**，这两个在一次安装后都会出现。

## 进行 SSH 连接
打开 **PuTTY**，输入 Host Name ( IP address )，和 Port  (一般为22) ，然后按下 **open**，进入一个命令行窗口。在命令行窗口中，输入登录信息（login as 与 password，这两个在购买 VPS 的时候就已经得到了）。
> 密码在输入的过程中，是不显示的，如果输入正确，会显示一些 VPS 系统相关的简要信息。

## 更新 VPS 的 packages
在上一步中如果密码输入正确，还会显示需要更新的内容。输入一下内容，进行可更新 packages 的检查。

```
apt update
```

再次输入 apt update，即可开始更新，更新完成后需要 reboot，使更新生效。

## 创建新用户并修改用户权限
前面都使用 root 账户操作，root 对系统具有全部权限，因而可能对系统造成严重的损害，所以使用 root 是 不够安全的。而一个具有 superuser 权限的常规账户，需要在命令前面加上 **sudo** 前缀，才能获取管理员权限。
添加新用户：

```
adduser <your new username>
```
为新用户增加 superuser 权限：
```
usermod -aG sudo <your new username>
```
## 增加 public key 认证
打开 **PuTTYgen**，点击 **generate** ，产生 public key（运行时间有点长，大概几十分钟），运行完成显示如下。需要填写 **Key passphrase**，passphrase 会与产生的 key 一起作为密码。

![Setting up a passphrase for your keys.](https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2018/10/passphrase.jpg)

填写完成后，在页面选择 **Save private key**，讲产生的文件保存在自己的电脑上。先不要关闭窗口，后面的操作还需要 **copy public key**。

按照前面2步的方法，使用原来的 **root** 账号登录VPS。使用以下命令，移动到之前创建的新用户的 home directory，这样命令行会对应到创建的新用户。
```
su – yournewusernam
```
![Switching users via the command line.](https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2018/10/switch-users.jpg)

之后，按照顺序输入下面的命令，作用是：为 **public key** 创建新文件夹，限制获取这个文件夹的权限，并且保存 public key。

```
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```
最后一个命令会开启 **Nano editor**。

![image-20220329162647759](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329162647759.png)

从之前的 **PuTTYgen** 窗口复制 **public key**，并且粘贴在这里。

> 一定要是直接从 Puttygen 复制，才符合authorized_keys的格式要求，后面才不会连接不上！如果发生了这样的事故，请在VPS运行商的页面上选择 Access > Reset root password

![image-20220329173447392](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329173447392.png)

然后按住 **CTRL + X**，来关闭 **Nano editor**。系统会询问是否保存对 **authorized_keys** 文件的更改，输入 **Y**，确认更改。

![image-20220329162814964](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329162814964.png)

输入下面的命令，作用：更改刚才编辑的文件的 permissions，并返回到 **root** 用户

```
chmod 600 ~/.ssh/authorized_keys
exit
```
打开 **PuTTY**，选择菜单 **Connection › SSH › Auth**，Browser 载入 private keys。

![image-20220329163301027](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329163301027.png)

通过SSH连接VPS，并且按照前面的方法，以新用户的身份登入。输入下面的命令来关闭默认的只有密码的认证方式。

```
sudo nano /etc/ssh/sshd_config
```

![image-20220329165515756](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329165515756.png)

修改完成后，和前面一样，按住 **CTRL + X**，来关闭 **Nano editor**，输入 **Y**，确认更改。**Reboot VPS**，下次登录的时候，就需要使用 **private key** 和 **passphrase** 了。

![image-20220329171546370](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329171546370.png)

## 为 VPS 安装 firewall
