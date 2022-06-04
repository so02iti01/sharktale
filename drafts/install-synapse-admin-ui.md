---
title: Ubuntu|安装 synapse-admin 用户管理界面
date: 2022-06-04 18:04:15
tags:
- Matrix
- Synapse
---

* 配置管理页面（来自[使用docker搭建Synapse[Matrix]](https://www.b612.me/code/223.html)）

riot(element)没有后台管理界面，可以使用synapse-admin搭建一个管理页面

```dockerfile
docker run -d -p 8100:80 awesometechnologies/synapse-admin
```
在之前的NGINX配置中添加

```nginx
location /admin/ {
        proxy_pass http://localhost:8100/; # 127.0.0.1:
        proxy_set_header X-Forwarded-For $remote_addr;
}
```

* 根据官方步骤2）没有做出来

```bash
yarn start
```

> Compiled successfully!
>
> You can now view synapse-admin in the browser.
>
>   http://localhost:3000
>
> Note that the development build is not optimized.
> To create a production build, use yarn build.
`ctrl+c` 退出这个界面
