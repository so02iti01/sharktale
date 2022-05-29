---
title: Win|安装 texlive2022 和 texstudio
date: 2022-04-08 12:50:36
tags: 
- texlive
- texstudio
---
以前有用过 `TeXLive2020`，现在为了做一个 `todolist`，加上换了电脑，所以重新安装了 `TeXLive` 和 `TeXStudio`
<!-- more -->

解压缩 `texlive2022-20220321.iso`，找到并打开`tl-tray-menu.exe`，在任务栏中找到对应图标，右键选择`Command Prompt`
执行下面的命令：
```bash
install-tl-windows -repository https://mirrors.tuna.tsinghua.edu.cn/tlpretest/ -gui
```
![ScreenShot_20220408125519](D:\Sharktale\source\images\ScreenShot_20220408125519.jpeg)
选择local repository:
![ScreenShot_20220408125941](D:\Sharktale\source\images\ScreenShot_20220408125941.jpeg)
选择安装位置，以及是否安装 TexWorks 前端，之后进入安装。这里由于不习惯 TeXworks，所以直接没有安装 TeXworks。
![ScreenShot_20220408130135](D:\Sharktale\source\images\ScreenShot_20220408130135.jpeg)
完成安装 `TeXLive2022`，安装 `TeXStudio` 就是一般软件的安装方法，不需要更多描述。
![ScreenShot_20220408144334](D:\Sharktale\source\images\ScreenShot_20220408144334.jpeg)
