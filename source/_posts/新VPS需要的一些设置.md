---
title: 新VPS需要的一些设置
date: 2022-03-29 15:18:29
updated: 2022-03-30 19:48:15
tags: VPS
---

给自己记个笔记......万一那一天需要重新做呢。
<!-- more -->

主要参考：[5 Steps to Get Your New Virtual Private Server (VPS) Ready to Use](https://www.hostinger.com/tutorials/getting-started-with-vps-hosting)

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
打开 **PuTTYgen**，点击 **generate** ，产生 public key，运行完成显示如下。需要填写 **Key passphrase**，passphrase 会与产生的 key 一起作为密码。

> keys 产生速度慢的原因找到了......是没有再产生时，在下方空白随意移动鼠标（提示文字说这样可以加入随意的部分在keys中）

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

通过SSH连接VPS，并且按照前面的方法，以新用户的身份登入。登录成功的话，会显示：

![image-20220330183914300](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220330183914300.png)

输入下面的命令来关闭默认的密码的认证方式。

> 注意：如果前面没有用keys登录成功，就不要开始关闭密码认证方式。如果前面设置错了，直接去服务商的网站上去 Rebuild > re-install，然后 Access >Reset root password

```
sudo nano /etc/ssh/sshd_config
```

![image-20220329165515756](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329165515756.png)

修改完成后，和前面一样，按住 **CTRL + X**，来关闭 **Nano editor**，输入 **Y**，确认更改。**Reboot VPS**，下次登录的时候，就需要使用 **private key** 和 **passphrase** 了。

> 这时候，如果open的时候没有加载 private keys，就会显示下面的信息。

![image-20220329171546370](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220329171546370.png)

## 为 VPS 安装 firewall

设置防火墙之后，可以限制 VPS 向网络开放的端口。这样可以阻止很多针对服务器的攻击。可以使用 [iptables](https://en.wikipedia.org/wiki/Iptables) ，来设置一个防火墙。这部分参考了[ptables Tutorial – Securing Ubuntu VPS with Linux Firewall](https://www.hostinger.com/tutorials/iptables-tutorial)。

> iptables 只适用于 **ipv4** 协议，如果需要适用 **ipv6**协议，需要转而使用**ip6tables**

### 什么是 iptables

iptables 是 Linux 的一种防火墙程序，它使用 tables 监控来自和去往你的服务器的流量。这些 tables 包含一系列的 rules (**chains**), 可以过滤包含数据的 packets。

[optin-monster-shortcode id=”fv4lqeko3gylvecpszws”]

每当一个 **packet** 符合一个 **rule**, 会被添加一个 **target**。 **target** 可以是另一个 **rule** 或者下列特殊值之一：

- **ACCEPT** – 允许通过
- **DROP** – 丢弃（不能通过）
- **RETURN** – 不能通过这一次的 rule，转送到上一次的 rule

默认的 tables (**filter**)包含下面3个 rules：

- **INPUT** – 过滤进入 server 的 packet

- **FORWARD** – 过滤将要被 server 转发 (forward) 的 packet

- **OUTPUT** – 过滤离开 server 的 packet

### 安装并使用 Iptables

#### 安装 Iptables

```
sudo apt-get update
sudo apt-get install iptables
```

检查现有 Iptables 设置：

```
sudo iptables -L -v
```

-L：列出所有的rules， -v：展示详细信息

输出结果如下，显示所有的 chains 都设置为了 **ACCEPT**，也即是没有 rules（所有的 packet 都能通过）。

![image-20220330191249042](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220330191249042.png)

#### 定义新 chain rules

新定义的 rule 需要被挂在 chain 的后面，所以需要在 iptables 命令后面加上 **-A** 选项 (**Append**)，同时与下面的选项结合：

- **-i** (**interface**) — the network interface whose traffic you want to filter, such as eth0, lo, ppp0, etc.
- **-p** (**protocol**) — the network protocol where your filtering process takes place. It can be either **tcp**, **udp**, **udplite**, **icmp**, **sctp**, **icmpv6**, and so on. Alternatively, you can type **all** to choose every protocol.
- **-s** (**source**) — the address from which traffic comes from. You can add a hostname or IP address.
- **–dport** (**destination port**) — the destination port number of a protocol, such as **22** (**SSH**), **443** (**https**), etc.
- **-j** (**target**) — the target name (**ACCEPT**, **DROP**, **RETURN**). You need to insert this every time you make a new rule.

```
sudo iptables -A <chain> -i <interface> -p <protocol (tcp/udp) > -s <source> --dport <port no.>  -j <target>
```

在下面用 **INPUT** chain 作为示例。

##### 允许主机内通信

使用 lo (loopback) 接口：

```
sudo iptables -A INPUT -i lo -j ACCEPT
```

这个命令让同一台机器上的数据库和应用程序正常通信

##### 开启 HTTP, SSH 和 SSL 端口

协议和端口号的对应是：**http** (port **80**), **https** (port **443**), 和 **ssh** (port **22**) 。这里需要指定 **-p** 和 **–dport** 参数。 

```
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

检查是否添加进 rule：

```
sudo iptables -L -v
```

![The accepted destination port in iptables which consist of http, https, and ssh](https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2017/06/destination-port-iptables.png)

##### 基于 IP 地址过滤 packet

用到 **-s** 选项，例如扔掉来自 192.168.1.3 的 packet：

```
sudo iptables -A INPUT -s 192.168.1.3 -j DROP
```

如果想要扔掉某个范围 IP地址的 packet，需要先加上 **-m**  **iprange**，然后用 **––src-range** 加上 IP 地址的范围，例如：

```
sudo iptables -A INPUT -m iprange --src-range 192.168.1.100-192.168.1.200 -j DROP
```

##### 扔掉不符合端口号的所有packets

需要先用上面的方法，设置允许的端口号。

```
sudo iptables -A INPUT -j DROP
```

##### 删除 rules

**删除所有的rules**：**-F** 选项 (**flush**):

```
sudo iptables -F
```

**删除某一条rule**： -D 选项：

先排序查看 rules

```
sudo iptables -L --line-numbers
```

会出现类似下面的情况

```
Chain INPUT (policy ACCEPT)

num  target     prot opt source               destination

1    ACCEPT     all -- 192.168.0.4          anywhere
2    ACCEPT     tcp -- anywhere             anywhere tcp dpt:https
3    ACCEPT     tcp -- anywhere             anywhere tcp dpt:http
4    ACCEPT     tcp -- anywhere             anywhere tcp dpt:ssh
```

需要用到 chain 类型和序号来执行删除命令。例如，删除 **INPUT** chain 的第 3 条：

```
sudo iptables -D INPUT 3
```

#### 关闭 Iptables 防火墙

```
sudo iptables -F
sudo /sbin/iptables-save
```

![The results after making changes persistent in iptables](https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2017/06/iptables-persistent-changes.png)

#### 保存 Iptables 更改

Iptables chains 更改的数据存于缓存里，但是重启 server 之后需要重新定义 chains。采用下面的命令，保证重启后更改仍是生效的。

> 每次更改 Iptables 后，都应该运行这个命令

```
sudo /sbin/iptables-save
```



