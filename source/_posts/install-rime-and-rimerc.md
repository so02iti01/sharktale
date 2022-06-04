---
title: Win|安装 rimerc 
date: 2022-04-27 22:16:25
updated:
tags: 
- rime
- rimerc
- 输入法
---

记录安装 rime 输入法和 rimec 配置文件的过程

<!--more-->

Rime 输入法是一个开源输入法，Rimerc 则解决了 Rime 新手配置难题，开箱即用的特点为输入法用户带来新的可能。本次安装参考 [Rimerc](https://github.com/Bambooin/rimerc) (rimer's dictionary & config) 的官方说明。

## 安装 rime 输入法

- Android 系统在 F-driod 上搜索 [Trime](https://f-droid.org/packages/com.osfans.trime/)，即可安装完成。
- windows 系统 在[小狼毫](https://rime.im/)的网页上下载，并安装。
- 其余系统参考 [rimerc](https://github.com/Bambooin/rimerc) 的 Readme> Usage > [Path](https://github.com/Bambooin/rimerc#path) 。

## 下载并安装 rimerc 配置文件

这里我采用的是[手动安装](https://github.com/Bambooin/rimerc#manual)。

1. 在 [release](https://github.com/Bambooin/rimerc/releases) 下载最新版本并解压缩，得到一个名为 rime 的文件夹

> - Android 需要下载 [rimerc-trime-0.1.6.zip](https://github.com/Bambooin/rimerc/releases/download/0.1.6/rimerc-trime-0.1.6.zip)，Windows 需要下载 [rimerc-weasel-0.1.6.zip](https://github.com/Bambooin/rimerc/releases/download/0.1.6/rimerc-weasel-0.1.6.zip)，其余系统需要下载的是带有不同输入法名字的.zip文件（名字参照安装 rime 输入法的名字）
> - 如果 Android 不好稳定连接 Github 的话，可以先下载到电脑上，然后通过 KDE Connect 这一类的软件，传输给手机

2. 把解压缩得到的 rime 的文件夹的全部内容，存放到指定的位置

> - Android 放在 rime 文件夹里（在所有文件/rime）
> - Windows 放在 C:\Users\ (你的用户名)\\AppData\Roaming\Rime 里

3. 找到输入法设定，选择使用的输入法

> 词库和配置文件，包含明月拼音，仓颉五代等。
>
> 如果是使用简体字，推荐选择 `明月拼音简化字`
