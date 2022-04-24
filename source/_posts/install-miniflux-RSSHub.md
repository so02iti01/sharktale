---
title: 安装和使用 miniflux
date: 2022-04-24 20:04:27
tags:
- miniflux
- RSShub
- RSS
---

> 安装部分参考 [Docker 快速搭建 Miniflux + RSSHub ](https://docs.rsshub.app/)，讲解细致，全程复制粘贴就可以完成！所以不需要我多写啦。

这里主要想记录一下使用 `miniflux + RSSHub` 进行 `RSS订阅` 的笔记~

## 一般内容的 RSS 订阅

1. 首先，在 miniflux 选择 Feeds > Add subscription 

2. 然后在 [RSSHub 官网](https://docs.rsshub.app/) 查找**路由**

   > 打开官网需要科学

3. 根据前面提到的教程的介绍，在**路由**前面加上 `http://rsshub:1200` ，得到 URL

   - 如果路由为 `/bilibili/user/followers/:uid`
   - 那么获得的 URL 格式为：`http://rsshub:1200`/bilibili/user/followers/:uid

4. 把得到的 URL 填入 miniflux 的订阅界面

## miniflux 抓取全文

在 miniflux 选择 Feeds > 一个订阅的空间 > Edit > `Fetch original content` (需要页面拉到最下面)

## 其余（Newletter）订阅

这部分由于还没开始用，所以就没写。用到哪里，写到哪里~
