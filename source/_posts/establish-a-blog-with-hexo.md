---
title: Hexo|blog 建立与使用笔记
date: 2022-03-25 13:58:04
updated:
tags: 
  - Hexo
  - git
---

## 建立Hexo blog

我在做下面这些之前，对这些都一无所知，过程磕磕绊绊，所以更加想要记录下来。

### Hexo与模板的安装

#### 开始Hexo之前的一些必要安装

- git
- node.js

#### 配置GitHub Page

先写使用GitHub自带的域名的步骤，如果要换成自己的域名，需要多进行一些写在后面的步骤。

## 使用Hexo blog

Hexo建议采用2个branch，一个放置博客源代码，一个放置产生的静态文件。所以我的博客仓库有2个分支，blog 分支和 master 分支。

hexo命令：

```
 $ hexo <command>
```

如果输入不存在的`command`（比如：hexo -s），会显示出hexo相关命令的帮助。

生成一个名为<filename>的，新的markdown文件（位于\source\_posts\）：

```
$ hexo new "<filename>"
```

> 注意：文章内部不要使用 H1 标题。

编译形成网页（使用http://localhost:4000/访问本地主机，即可看到效果） ：

```
$ hexo s
```

git命令多看[git cheat sheet.pdf](https://training.github.com/downloads/zh_CN/github-git-cheat-sheet/)，外加多搜索

日常修改与版本控制

1. 将Hexo配置推到remote的branch

```
$ git add .               
$ git commit -m "[descriptive messgage]"  
$ git push origin Hexo    # 将本地branch的内容全部上传到与之关联的remote branch
```

> 说明：
>
> `$ git add`			Snapshots the file in preparation for versioning
>
> `$ git commit -m "<descriptive messgage>"` 			Records file snapshots permanently in version history
>
> `$ git push`			upload all local branch commits to [GitHub](https://github.com/)
>
> `git`			an open source, distributed version-control system
>
> `commit`			a Git object, a snapshot of your entire repository compressed into a SHA

2. 把文章推到master分支

```
$ hexo g -d
```

> `$ hexo g`	Generate static files, options: 
>
> | Command               | Description                                                  |
> | :-------------------- | ------------------------------------------------------------ |
> | -b 或者 --bail        | Raise an error if any unhandled exception is thrown during generation |
> | -c 或者 --concurrency | Maximum number of files to be generated in parallel. Default is infinity |
> | `-d 或者 --deploy`    | `Deploy after generated`                                     |
> | -f 或者--force        | Force regenerate                                             |
> | -w 或者--watch        | Watch file changes                                           |
>



------

## 参考资料

[1] Barrel Titor, [用 Hexo 和 GitHub Page 搭建静态博客](https://zhuanlan.zhihu.com/p/149531391#:~:text=%E5%9C%A8%20GitHub%20Page%20%E4%B8%8A%E6%9C%89%E4%B8%A4%E4%B8%AA%E4%B8%BB%E6%B5%81%E7%9A%84%E9%9D%99%E6%80%81%E5%8D%9A%E5%AE%A2%E6%A1%86%E6%9E%B6%EF%BC%9AJekyll%20%E5%92%8C%20Hexo%E3%80%82%20Jekyll,%E4%B8%8E%20GitHub%20%E5%A5%91%E5%90%88%E5%BA%A6%E6%9B%B4%E9%AB%98%EF%BC%8C%E5%8F%AF%E4%BB%A5%E5%9C%A8%20GitHub%20%E4%B8%AD%E7%9B%B4%E6%8E%A5%E7%94%9F%E6%88%90%E9%A1%B5%E9%9D%A2%EF%BC%9BHexo%20%E4%B8%AD%E6%96%87%E8%B5%84%E6%96%99%E6%9B%B4%E4%B8%B0%E5%AF%8C%EF%BC%8C%E5%9F%BA%E7%A1%80%E5%8A%9F%E8%83%BD%E6%9B%B4%E5%A4%9A%EF%BC%8C%E4%BD%86%E6%98%AF%E9%83%A8%E7%BD%B2%E5%92%8C%E5%A4%87%E4%BB%BD%E6%AF%94%20Jekyll%20%E9%BA%BB%E7%83%A6%E3%80%82), 知乎. (最后一次获取时间：2022-03-25)

