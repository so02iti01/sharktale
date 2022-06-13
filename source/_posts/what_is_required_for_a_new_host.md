---
title: Ubuntu|新 linux 主机需要的一些设置
date: 2022-05-18 13:06:39
updated:
tags: 
- VPS
- PuTTY
- Iptables
---

updated （2022-05-18）: 根据 [酸橘汁腌鱼](https://seviche.cc/) 的 [VPS 安全初始化 ](https://seviche.cc/2022-05-07-vps-init) 进行了修改

<!-- more -->

## 设置与连接 SSH 
###  安装 PuTTY 
使用 windows 电脑连接 remote 的 Linux VPS，需要一个图形化的界面

>  If your Windows is Windows 10 Version 1803, OpenSSH Client has been implemented as a Windows feature, so it's possible to use ssh command on command prompt without Putty and other SSH software.

open SSH 客户端的使用大同小异。下载 [PuTTY](https://www.putty.org/)，本次需要用到的是 `PuTTY` 和 `PuTTYgen`，这两个在一次安装后都会出现

### 进行 SSH 连接
打开 `PuTTY`，输入 Host Name ( IP address )，和 Port  (一般为22) ，然后按下 `open`，进入一个命令行窗口。在命令行窗口中，输入登录信息（login as 与 password，这两个在购买 VPS 的时候就已经得到了）
> 密码在输入的过程中，是不显示的，如果输入正确，会显示一些 VPS 系统相关的简要信息

### 更新 VPS 的 packages
在上一步中如果密码输入正确，还会显示需要更新的内容。输入一下内容，进行可更新 packages 的检查

```shell
apt update
```

再次输入 apt update，即可开始更新，更新完成后需要 reboot，使更新生效

### 创建新用户并修改用户权限
前面都使用 root 账户操作，root 对系统具有全部权限，因而可能对系统造成严重的损害，所以使用 root 是 不够安全的。而一个具有 superuser 权限的常规账户，需要在命令前面加上 `sudo` 前缀，才能获取管理员权限。
添加新用户：

```shell
adduser <your new username>
```
为新用户增加 superuser 权限：
```shell
usermod -aG sudo <your new username>
```
### 增加 public key 认证
打开 `PuTTYgen`，点击 `generate` ，产生 public key。需要填写 `Key passphrase`，passphrase 会与产生的 key 一起作为密码

> keys 产生速度慢的原因找到了......是没有再产生时，在下方空白随意移动鼠标（提示文字说这样可以加入随意的部分在keys中）

填写完成后，在页面选择 `Save private key`，把产生的文件保存在自己的电脑上。先不要关闭窗口，后面的操作还需要 `copy public key`

按照前面2步的方法，使用原来的 `root` 账号登录VPS。

使用以下命令，可以移动到之前创建的新用户的 home directory，这样命令行会对应到创建的新用户

```shell
su – <your new username>
```
之后，按照顺序输入下面的命令，作用是：为 `public key` 创建新文件夹，限制获取这个文件夹的权限，并且保存 public key

```shell
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```
最后一个命令会开启 `Nano editor`。

从之前的 `PuTTYgen` 窗口复制 `public key`，并且粘贴在这里。

> 一定要是直接从 Puttygen 复制，才符合authorized_keys的格式要求，后面才不会连接不上！如果发生了这样的事故，请在VPS运行商的页面上选择 Access > Reset root password

然后按住 `CTRL + X`，来关闭 `Nano editor`。系统会询问是否保存对 `authorized_keys` 文件的更改，输入 `Y`，确认更改

输入下面的命令，作用：更改刚才编辑的文件的 permissions，并返回到  `root`  用户

```shell
chmod 600 ~/.ssh/authorized_keys
exit
```
打开 `PuTTY`，选择菜单 `Connection › SSH › Auth`，Browser 载入 private keys

通过SSH连接VPS，并且按照前面的方法，以新用户的身份登入

输入下面的命令来关闭默认的密码的认证方式

> 注意：如果前面没有用keys登录成功，就不要开始关闭密码认证方式。如果前面设置错了，直接去服务商的网站上去 Rebuild > re-install，然后 Access >Reset root password，然后一切重头再来......

```shell
sudo nano /etc/ssh/sshd_config
```

修改下面的内容（这里参考了鱼的文章）

```
PermitRootLogin no    
AllowUsers username   #如果没有这一行就手动添加
RSAAuthentication yes #这一行我找不到就没有配置
PubkeyAuthentication yes
PasswordAuthentication no # 禁止使用密码登录
```

修改完成后，和前面一样，按住 `CTRL + X`，来关闭 `Nano editor`，输入 `Y`，确认更改

`Reboot VPS` 或者 输入 

```shell
service sshd restart
```

下次登录的时候，就需要使用 `private key` 和 `passphrase` 了

### 改 SSH 端口

这部分是新增的，参考了鱼的文章

还是在 `/etc/ssh/sshd_config`中，在 `Port 22` 下面增加一行

```
Port <你选择的端口号>     # 换一个22以外的端口号
```

重启 sshd

```shell
sudo service sshd restart
```

打开防火墙，为新增的端口放行。设置好之后，用新端口重新连接一下，没问题的话，注释掉 `Port 22` 

## 为 VPS 安装防火墙

设置防火墙之后，可以限制 VPS 向网络开放的端口。这样可以阻止很多针对服务器的攻击。可以使用 [iptables](https://en.wikipedia.org/wiki/Iptables) ，来设置一个防火墙。

> iptables 只适用于 `ipv4` 协议，如果需要适用 `ipv6`协议，需要转而使用`ip6tables`

`ufw` 也是一个常用的 firewall，也可以使用 `ufw` （`ufw` 的设置说不定还要更简单

### 什么是 iptables

iptables 是 Linux 的一种防火墙程序，它使用 tables 监控来自和去往你的服务器的流量。这些 tables 包含一系列的 rules (`chains`), 可以过滤包含数据的 packets。

[optin-monster-shortcode id=”fv4lqeko3gylvecpszws”]

每当一个 `packet` 符合一个 `rule`, 会被添加一个 target。 target 可以是另一个 `rule` 或者下列特殊值之一：

- `ACCEPT` – 允许通过
- `DROP` – 丢弃（不能通过）
- `RETURN` – 不能通过这一次的 rule，转送到上一次的 rule

默认的 tables (`filter`)包含下面3个 rules：

- `INPUT` – 过滤进入 server 的 packet

- `FORWARD` – 过滤将要被 server 转发 (forward) 的 packet

- `OUTPUT` – 过滤离开 server 的 packet

### 安装并使用 Iptables

#### 安装 Iptables

```shell
sudo apt-get update
sudo apt-get install iptables
```

检查现有 Iptables 设置：

```shell
sudo iptables -L -v
```

-L：列出所有的rules， -v：展示详细信息

显示所有的 chains 都设置为了 `ACCEPT`，也即是没有 rules（所有的 packet 都能通过）。

#### 定义新 chain rules

新定义的 rule 需要被挂在 chain 的后面，所以需要在 iptables 命令后面加上 `-A` 选项 (`Append`)，同时与下面的选项结合：

- `-i (interface)` — the network interface whose traffic you want to filter, such as eth0, lo, ppp0, etc.
- `-p (protocol)` — the network protocol where your filtering process takes place. It can be either `tcp`, `udp`, `udplite`, `icmp`, `sctp`, `icmpv6`, and so on. Alternatively, you can type `all` to choose every protocol.
- `-s (source`) — the address from which traffic comes from. You can add a hostname or IP address.
- `–dport (destination port)` — the destination port number of a protocol, such as `22 (SSH)`, `443 (https)`, etc.
- `-j (target)` — the target name (`ACCEPT`, `DROP`, `RETURN`). You need to insert this every time you make a new rule.

```shell
sudo iptables -A <chain> -i <interface> -p <protocol (tcp/udp) > -s <source> --dport <port no.>  -j <target>
```

在下面用 `INPUT` chain 作为示例。

##### 允许主机内通信

使用 lo (loopback) 接口：

```shell
sudo iptables -A INPUT -i lo -j ACCEPT
```

这个命令让同一台机器上的数据库和应用程序正常通信

##### 开启 HTTP, SSH 和 SSL 端口

协议和端口号的对应是：`http` (port `80`), `https` (port `443`), 和 `ssh` (port `22`) 。这里需要指定 `-p` 和 `–dport` 参数。 

```shell
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

检查是否添加进 rule：

```shell
sudo iptables -L -v
```

##### 基于 IP 地址过滤 packet

用到 `-s` 选项，例如扔掉来自 192.168.1.3 的 packet：

```shell
sudo iptables -A INPUT -s 192.168.1.3 -j DROP
```

如果想要扔掉某个范围 IP地址的 packet，需要先加上 `-m  iprange`，然后用 `––src-range` 加上 IP 地址的范围，例如：

```shell
sudo iptables -A INPUT -m iprange --src-range 192.168.1.100-192.168.1.200 -j DROP
```

##### 扔掉不符合端口号的所有packets

需要先用上面的方法，设置允许的端口号。

```shell
sudo iptables -A INPUT -j DROP
```

##### 删除 rules

**删除所有的rules**：`-F` 选项 (`flush`):

```shell
sudo iptables -F
```

**删除某一条rule**： `-D` 选项：

先排序查看 rules

```shell
sudo iptables -L --line-numbers
```

会出现类似下面的情况

```bash
Chain INPUT (policy ACCEPT)

num  target     prot opt source               destination

1    ACCEPT     all -- 192.168.0.4          anywhere
2    ACCEPT     tcp -- anywhere             anywhere tcp dpt:https
3    ACCEPT     tcp -- anywhere             anywhere tcp dpt:http
4    ACCEPT     tcp -- anywhere             anywhere tcp dpt:ssh
```

需要用到 chain 类型和序号来执行删除命令。例如，删除 `INPUT` chain 的第 3 条：

```shell
sudo iptables -D INPUT 3
```

#### 关闭 Iptables 防火墙

```shell
sudo iptables -F
sudo /sbin/iptables-save
```

#### 保存 Iptables 更改

Iptables chains 更改的数据存于缓存里，但是重启 server 之后需要重新定义 chains。采用下面的命令，保证重启后更改仍是生效的。

> 每次更改 Iptables 后，都应该运行这个命令

```shell
sudo /sbin/iptables-save
```

## 进一步的安全防护
这里参考 [酸橘汁腌鱼](https://seviche.cc/) 的 [VPS 安全初始化 ](https://seviche.cc/2022-05-07-vps-init) ，感谢鱼的文章，让我知道了更多 VPS 的安全防护方法

###  运行防病毒软件 ClamAV

```shell
# 安装
sudo apt update
sudo apt install clamav clamav-daemon -y
sudo systemctl stop clamav-freshclam # 停止服务
sudo freshclam # 执行更新
sudo systemctl start clamav-freshclam # 再次启动 clamav-freshclam
sudo systemctl is-enabled clamav-freshclam # 设置开机自启动
ls /var/lib/clamav/ # 下载 ClamAV 数据库
# nice：降低 clamscan 的优先级（限制相对 cpu 时间）
sudo nice -n 15 clamscan # 限制 Clamscan CPU 使用率
# cpulimit：限制绝对的 CPU 时间。 安装cpulimit
sudo apt-get install cpulimit
cpulimit -z -e clamscan -l 20 & clamscan -ir /
```

其它可能用到的命令

```shell
clamscan /home/filename.docx  #扫描特定目录或文件
clamscan --no-summary /home/ #扫描结束时不显示摘要
clamscan -i / #打印受感染的文件
clamscan --bell -i /home #警惕病毒检测
clamscan -r --remove /home/USER #删除受感染的文件
```

返回码

- 0：未发现病毒。
- 1：发现病毒。
- 2：发生了一些错误。

### 安装 fail2ban 以阻止重复登录尝试

```shell
sudo apt update 
sudo apt upgrade -y
sudo apt install fail2ban
sudo nano /etc/fail2ban/jail.local
```

写入

```shell
[DEFAULT]
destemail = your@email.here
sendername = Fail2Ban

[sshd]
enabled = true
port = 22   # 换成前面自己设置的 SSH 端口号

[sshd-ddos]
enabled = true
port = 22  # 换成前面自己设置的 SSH 端口号
```

重启fail2ban

``` shell
sudo systemctl restart fail2ban
```

## 最后的最后

鱼的新文章还给出了安全检查的 tips，看一看会更有收获：

https://seviche.cc/2022-05-07-vps-init#%E6%97%A5%E5%B8%B8%E9%98%B2%E6%8A%A4

## 参考资料

[1] [5 Steps to Get Your New Virtual Private Server (VPS) Ready to Use](https://www.hostinger.com/tutorials/getting-started-with-vps-hosting)

[2] [Iptables Tutorial – Securing Ubuntu VPS with Linux Firewall](https://www.hostinger.com/tutorials/iptables-tutorial)

[3] [酸橘汁腌鱼](https://seviche.cc/) 的 [VPS 安全初始化 ](https://seviche.cc/2022-05-07-vps-init) 

