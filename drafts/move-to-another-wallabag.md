---
title: wallabag|用户的实例间搬迁
date: 2022-06-13 10:19:20
tags:
- wallabag
---

  本说明针对 wallabag V2 搬迁至 wallabag V2

<!--more-->

# 在开始搬家的地方：导出 All articles.json

在旧的 wallabag 中选择 `Export`

![image-20220613101602281](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220613101602281.png)

选择 `JSON`

![image-20220613101643538](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220613101643538.png)

之后会下载出一个名为 `All articles.json` 的文件，保存好

# 在搬家的目的地：导入 All articles.json

在新的 wallabag 实例中，点击右上角的 `My account`，选择 `Import`

![image-20220613101528968](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220613101528968.png)

选择 `wallabag V2` 分类下面的 `IMPORT CONTENTS`

![image-20220613102350263](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220613102350263.png)

点击 `FILE`，选择在旧实例导出的 `All articles.json` ，然后就是点击 `UPLOAD FILE`，之后就是静静等着数据的传输

![image-20220613102429502](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220613102429502.png)

# 清除旧帐号的信息

打开旧的 wallabag，右上角 `My account` —> `Config` —>最后一栏 `RESET AREA`

点击 `REMOVE ALL ENTRIES` 和 最下面的`DELETE MY ACCOUNT`

![image-20220613102846031](C:\Users\19914\AppData\Roaming\Typora\typora-user-images\image-20220613102846031.png)