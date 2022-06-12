---
title: Notes|docker 使用笔记
date: 2022-06-12 18:09:13
tags:
- docker
- docker compose
---

写一个我会用到的关于 docker 和 docker compose 的简单笔记

<!--more-->

## 容器上线与删除

```bash
sudo nano docker-compose.yml      # 写 dockerfile    
sudo rm docker-compose.yml        # 移除 dockerfile
docker-compose up -d              # 上线容器
docker-compose down               # 删除容器
```

## 检查 docker 运行问题

针对容器


```bash
docker container ls               # 显示 container ID, 创建时间, 运行状态, ports
docker logs <container ID>        # 显示某个 container 的运行日志
```

针对镜像

```bash
docker images                     # 展示 IMAGE ID，IMAGE 的版本(时间)
docker rmi <IMAGE ID>             # 删除对应镜像
rm -rf miniflux-db                # 删除持久化数据
```
