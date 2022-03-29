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
下载 [PuTTY](https://www.putty.org/)，本次需要用到的是 **PuTTY** 和 **PuTTYgen**，这两个在一次安装后都会出现。

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
前面都使用 root 账户操作，root 对系统具有全部权限，因而可能对系统造成严重的损害，所以使用 root 是 不够安全的。而一个具有 superuser 权限的常规账户，需要在命令前面加上** sudo **前缀，才能获取管理员权限。
添加新用户：
```
adduser <your new username>
```
为新用户增加 superuser 权限：
```
usermod -aG sudo <your new username>
```
## 增加 public key 认证
打开 **PuTTYgen**，点击** generate**，产生 public key（运行时间有点长）。
