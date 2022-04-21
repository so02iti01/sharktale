---
title: 安装 element-web
date: 2022-04-21 22:18:40
tags: 
- element
- matrix
---

在上次的 synapse 上，添加了 element web

<!--more-->

```
wget https://github.com/vector-im/element-web/releases/download/v1.10.9-rc.3/element-v1.10.9-rc.3.tar.gz
sudo tar -zxvf element-v1.10.9-rc.3.tar.gz -C /etc/matrix-synapse/element
sudo mv /etc/matrix-synapse/element/element-v1.10.9-rc.3 /etc/matrix-synapse/element-web
cd /etc/matrix-synapse/element-web
sudo cp config.sample.json config.json
cd ~
sudo nano /etc/matrix-synapse/element-web/config.json
```

> 1. https://github.com/vector-im/element-web/releases/download/v1.10.9-rc.3/element-v1.10.9-rc.3.tar.gz 需要换成最新的element-web下载地址
> 2. /etc/matrix-synapse/element 和 /etc/matrix-synapse/element-web 是自己建立的文件夹

在”m.homeserver”中，修改”base_url”和”server_name”，适配自己的 matrix 服务器

在之前的 NGINX 配置文件中，添加

```
root /etc/matrix-synapse/element-web;
```

> 注意：需要添加到location上面
