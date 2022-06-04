---
title: MATLAB|从 IEEE Xplore 批量下载论文
date: 2022-05-24 14:55:39
updated:
tags: 
- download
- IEEE
- MATLAB
---

手动从 IEEE Xplore 导出 .csv，主要使用 MATLAB 的函数 `webread` 和 `websave` 实现自动批量下载。

<!--more-->

参考了[这篇](https://zouyuze.blogspot.com/2017/10/download-ieee-xplore-pdfs-in-batch.html)，修改了自定义函数的部分文字输出，写了些注释和使用说明

使用要求：

* 连接校园网
* 自己的校园网可以下载  IEEE Xplore
* 有 MATLAB

选好文献，导出 `.csv 文件`之后，填写 `.csv 文件`的位置

```matlab
clc
export = 'C:\Users\xxxx\Downloads\export2022.05.24-01.31.59.csv'; % .csv地址
skip = 0;
DownloadPDFfromXplore(export, skip); % pdf会下载到current folder
```

其中，自定义函数 `DownloadPDFfromXplore` 内容

```matlab
function DownloadPDFfromXplore(export, skip)
%% download pdf from IEEE EXplore export file
% if termites at any exception, we can restart and skip those downloaded 
if nargin == 1
    skip = 0;
end
[raw_numerical, raw_text, ~] = xlsread(export); % 读取.csv文件信息
UrlList = raw_text(skip+3:end, 16);
NameList = raw_text(skip+3:end, 1);
YearList = raw_numerical(skip+1:end, 1);
pat = '[\\/:*?"<>|]'; 
NameList = regexprep(NameList, pat, ' '); % 替换文件名中不允许出现的符号为空格
for k = 1 : length(NameList)
   html = webread(UrlList{k});
   first = strfind(html, '<iframe src="h');
   last = strfind(html, '" frameborder=0>');
   url = html(first+13:last-1);
   filename = [num2str(YearList(k)) ' ' NameList{k} '.pdf'];
   websave(filename, url); % 将url地址内容保存到本地并命名为filename。
   myString  = fprintf('(%d/%d) %s ... DONE\n',k,length(NameList),filename);
   disp(myString);  % 显示获取进度
   waitTime = 30 * rand() + 30;  % 模拟人工操作的随机等待时间，避免被封IP
   pause(waitTime);
end
```

