---
title: Notes|docker 使用与排查笔记
date: 2022-06-12 18:09:13
tags:
- docker
- docker compose
- nextcloud
---

我会用到的关于 docker 和 docker compose 的简单笔记

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
```

> 可以用这个命令，查看 container 是否在发生不断 restart 的故障

```bash
docker logs <container_ID>        # 显示某个 container 的运行日志
```

> 用来查看 container 发生故障的详细原因

```bash
docker stop <container_ID>       # 停止某个容器
docker rm <Container_ID>         # 删除某个容器
```

针对镜像

```bash
docker images                     # 展示 IMAGE ID，IMAGE 的版本(时间)
docker rmi <IMAGE_ID>             # 删除对应镜像
rm -rf miniflux-db                # 删除持久化数据，使用volumes:下面的名字
```

## 在运行的容器内部执行命令

非 docker 安装时，可以切换到某个目录，再执行这个应用特有的命令。但是，使用 docker 安装的应用，执行特定操作时，则需要用到 [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec/)命令

两种情况的情况的区别

```bash
# Disabling serverside encryption)
# (1)非 docker 安装的 Nextcloud
cd /var/www/nextcloud.
sudo -u www-data ./occ maintenance:singleuser –on.  # Switch the Nextcloud single user mode to on
sudo -u www-data ./occ encryption:disable. # Disable encryption
sudo -u www-data ./occ maintenance:singleuser –off  # Turn off single user mode with the command
# (2)docker 安装的 Nextcloud
docker exec -it -u www-data nextcloud php occ encryption:decrypt-all xx  # 一个例子
```

> 对比可以看出：（2）是在（1）的命令前面加了 `docker exec -it `

`docker exec ` 的用法cd

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

> options：
>
> | Name, shorthand        | Default | Description                                          |
> | ---------------------- | ------- | ---------------------------------------------------- |
> | `--detach` , `-d`      |         | Detached mode: run command in the background         |
> | `--detach-keys`        |         | Override the key sequence for detaching a container  |
> | `--env` , `-e`         |         | Set environment variables                            |
> | `--env-file`           |         | Read in a file of environment variables              |
> | `--interactive` , `-i` |         | Keep STDIN open even if not attached                 |
> | `--privileged`         |         | Give extended privileges to the command              |
> | `--tty` , `-t`         |         | Allocate a pseudo-TTY                                |
> | `--user` , `-u`        |         | Username or UID (format: <name\|uid>[:<group\|gid>]) |
> | `--workdir` , `-w`     |         | Working directory inside the container               |

examples：

```bash
# (1)create a container named ubuntu_bash and start a Bash session.
docker run --name ubuntu_bash --rm -i -t ubuntu bash
# (2)create a new file /tmp/execWorks inside the running container ubuntu_bash, in the background.
docker exec -d ubuntu_bash touch /tmp/execWorks
# (3)create a new Bash session in the container ubuntu_bash.
docker exec -it ubuntu_bash bash
# (4)create a new Bash session in the container ubuntu_bash with environment variable $VAR set to “1”.
docker exec -it -e VAR=1 ubuntu_bash bash     
# Note that this environment variable will only be valid on the current Bash session. By default docker exec command runs in the same working directory set when container was created.
```
