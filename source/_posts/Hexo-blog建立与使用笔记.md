---
title: Hexo blog建立与使用笔记
date: 2022-03-25 13:58:04
tags: Hexo;git
---

# 建立Hexo blog

我在做下面这些之前，对这些都一无所知，过程磕磕绊绊，所以更加想要记录下来。

## Hexo与模板的安装

### 开始Hexo之前的一些必要安装

- git bash
- node.js

# 使用Hexo blog

Hexo采用2个branch

hexo命令：

```
 $ hexo <command>
```

如果输入不存在的**command**（比如：hexo -s），会显示出hexo相关命令的帮助。

git命令多看**git cheat sheet.pdf**，外加多搜索

日常修改与版本控制

1. 将Hexo配置推到remote的branch

```
$ git add .               
$ git commit -m "<descriptive messgage>"  
$ git push origin Hexo    # 将本地branch的内容全部上传到与之关联的remote branch
```

说明：

**$ git add**	Snapshots the file in preparation for versioning

**$ git commit -m "<descriptive messgage>"** 	Records file snapshots permanently in version history

**$ git push**	upload all local branch commits to [Github](https://github.com/)

**git**	an open source, distributed version-control system

**commit**	a Git object, a snapshot of your entire repository compressed into a SHA

2. 把文章推到master分支

```
$ hexo g -d
```

**$ hexo g**	Generate static files, options: 

| Command               | Description                                                  |
| :-------------------- | ------------------------------------------------------------ |
| -b 或者 --bail        | Raise an error if any unhandled exception is thrown during generation |
| -c 或者 --concurrency | Maximum number of files to be generated in parallel. Default is infinity |
| **-d 或者 --deploy**  | **Deploy after generated**                                   |
| -f 或者--force        | Force regenerate                                             |
| -w 或者--watch        | Watch file changes                                           |

