---
title: 在 WSL 上 Emacs 的安装与使用
date: 2022-05-11 17:26:41
author: Southfox
tags:
- WSL
- Emacs
- Doom Emacs
---

可以在 Windows 系统中使用 `Windows Subsystem for Linux (WSL)`，来搭载 `Emacs` 和 `Doom Emacs`。

但是，千万不要直接 `sudo apt install emacs27`   ，安装的是 `emacs 26.3`，用不了 `doom emacs` 。操作根据 `doom emacs` 针对 wsl 的官方教程，在[这里](https://github.com/doomemacs/doomemacs/blob/master/docs/getting_started.org#with-wsl--ubuntu-1804-lts)。

狐狸对这个笔记提供了大量支持，加上我实现不了 coauthor 功能，所以这一篇的作者就写狐狸啦~

<!--more-->

## 安装 WSL，Emacs 和 Doom Emacs
### 安装 WSL (Ubuntu)
根据 Ubuntu 给出的 [Install WSL](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-11-with-gui-support#2-install-wsl)，按 Windows 键，搜索 `Windows features`，勾选 `Virtual Machine Platform`，开启自带的 虚拟机（也就是 WSL），重启电脑后生效

在 Microsoft store 中，搜索 `Ubuntu` 进行安装

> 如果选择名为  `Ubuntu` 的软件，就是安装最新的 `Ubuntu` 版本


安装结束后，打开 Ubuntu，升级 Ubuntu 的包                                               

   ```shell
   sudo apt update && sudo apt upgrade
   ```

### 安装 Emacs

   ```shell
   sudo add-apt-repository ppa:kelleyk/emacs
   sudo apt update
   sudo apt install emacs27                     
   ```

### 安装 Doom 的依赖                                                     

   ```shell
   # required dependencies
   sudo apt-get install git ripgrep
   # optional dependencies
   sudo apt-get install fd-find
   ```

### 安装 Doom Emacs

```shell
git clone https://github.com/hlissner/doom-emacs ~/.emacs.d
~/.emacs.d/bin/doom install
```

> 很容易因为网络，导致安装出问题，遇到了不要担心，多运行 `~/.emacs.d/bin/doom install` 几次就好啦

### 更新 Doom Emacs
这一步也可以先不做
```shell
~/.emacs.d/bin/doom sync       # 同步
~/.emacs.d/bin/doom upgrade    # 升级包
~/.emacs.d/bin/doom doctor     # 诊断问题
```
## 使用 Doom Emacs
安装好后用 `emacs -nw` 在终端里运行 `emacs`，`M-x` 然后输入 `load theme`  就能换主题了

> M-x 就是 Alt 键和 x 键同时按下

首先推荐用这个方式输入命令，他里面也会提示快捷键的，用的多了就能熟悉快捷键了~

提示了快捷键，`space-h-t` 也可以看主题

###  快捷命令

```
SPC .            # 打开文件菜单
SPC <            # 打开最近的窗口
bookmark set     # 将当前文件加入书签
SPC RET（空格）   # 打开收藏夹
SPC              # 退出窗口
SPC w h/j/k/l    # 就是管理窗口
SPC w q          # 退出现在聚焦的窗口
```

> SPC 是空格键 space
>
> 不过掌握最基础的 i 进入插入模式，然后 ESC 退出 :w 保存就行了

###  试试加包

`SPC f P` 是打开配置文件夹，选择 `init.el` 就可以编辑这个文件了。输入 `/org` 查找 org 字符串。按下`回车`就进入查找，移动到 `org 包`后按 `esc` 退出 查找模式，按下 `i` 进入插入模式，改成这样~

```elisp
156        (org
157          +roam2
158          +pomodoro)
```

> 这样就安装了 `roam2` 和 `番茄时钟` (可选)


按 `ESC` 退出 插入模式，按下 `:w` 回车保存文件。然后，`M-x` 输入 `reload` ，选择 `doom/reload` 重载配置，这样就会去安装包了


### 关于rime 的包

可以再安安 `rime` 的包，不过 `wsl` 里应该要 `apt install librime` 安一下依赖

这样就可以用上自己的设置了，不过不建议双方同步词库，比较难搞……

`emacs` 里的 `rime` 包好处就是有断言，输入中文打个空格可以自动切换英文

> emacs 下的 rime 路径是 `~/.emacs/.local/etc/rime/`

`vim 模式`有点不习惯也可以直接用 win 下的编辑器打开 `init.el` 文件，正好可以熟悉一下 `wsl` 里的文件放在哪个目录......毕竟要把自己的 `rime` 文件扔进 `wsl`  里……

`init.el` 文件里 `elisp 语言`的注释是 `;;` 

```
 (doom! :input
       ;;chinese ←这
       ;;japanese
       ;;layout            ; auie,ctsrnm is the superior home row
```


### 开始写笔记

`SPC .` 然后在一个路径下输入文件名，完成新建文件 (像 nano 一样)，就可以开始写笔记啦~

> 后缀名 .org 可以切到 org 模式哦

写日记需要的包是 `org-roam 包`

### 玩游戏

俄罗斯方块  `M-x tetris`，其余游戏见 [Fun and Games in Emacs](https://www.masteringemacs.org/article/fun-games-in-emacs)