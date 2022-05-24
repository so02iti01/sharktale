---
title: Hexo|关于 volantis 模板一些注意事项
date: 2022-03-25 15:20:22
tags: 
 - volantis
 - Hexo
---

## 导航栏设置

### 导航栏相关的页面布局设置

`归档`页面为自动生成，初始化时已经生成。`友链`，`关于`等页面都是需要写在`source/about/index.md`（如果没有需要创建一个）里面。

#### 关于页面

In `source/about/index.md`:

```
---
layout: docs
seo_title: 关于
bottom_meta: false
sidebar: []
---

下面写关于自己的内容
```

#### 分类页面

In `source/categories/index.md`:

```
---
layout: category
index: true
title: 所有分类
---
```

#### 标签页面

In  `source/tags/index.md`:

```
---
layout: tag
index: true
title: 所有标签
---
```

#### 友链页面

首先，在 `source/friends/index.md`写入：

```
---
layout: friends # 必须
title: 我的朋友们 # 可选，这是友链页的标题
---

这里写友链上方的内容。

<!-- more -->

这里可以写友链页面下方的文字备注，例如自己的友链规范、示例等。
```

插入友链数据可以 选择`布局方案`或[`使用友链标签`](https://volantis.js.org/v5/tag-plugins/#%E5%8F%8B%E9%93%BE%E6%A0%87%E7%AD%BE)。这里我选择布局方案。在主题配置文件中找到：

```
pages:
  # 友链页面配置
  friends:
    layout_scheme: traditional # simple: 简单布局（只有头像和标题）, traditional: 传统布局(Volantis 旧版本的风格)
```

在`/source/_data/friends.yml`（如果没有需要创建一个）写入友链数据，内容格式为：

```
- group: # 分组标题
  description:  # 分组描述
  items:
    - title: # 名称
      avatar: # 头像
      url: # 链接
      screenshot: # 截图
      keywords: # 关键词
      description: # 描述
    - title: # 名称
      avatar: # 头像
      url: # 链接
      screenshot: # 截图
      keywords: # 关键词
      description: # 描述
```

不同的布局方式，会用到一部分的字段，一般来说，`title`、`avatar` 和 `url` 都是必须的。这些数据被转成 HTML 标签插入到友链页面的 `<!-- more -->` 部分。

### 开启网站的站内搜索

`search`功能，默认由hexo自己产生，需要先执行下面的命令，安装必要插件：

```
$ npm i hexo-generator-json-content
```

## 网站图标

设置网站图标，需要在博客根目录下的 `_config.yml` 文件写入：

```
# 网站图标，更多尺寸等图标请使用import方式批量导入
favicon: <your icon url>
```

## Others

### 文章页面设置（Front-matter）

[**Front-matter**](https://hexo.io/zh-cn/docs/front-matter) 是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量，举例来说：

```
---
title: Hello World
date: 2013/7/13 20:46:25
---
```

分类，标签和各个参数值的详细介绍，见[**Front-matter**](https://hexo.io/zh-cn/docs/front-matter) 和[Volantis: 页面配置](https://volantis.js.org/v5/page-settings)。这里选取一些我会用到的。

在文章`Front-matter`之后，插入 `<!-- more -->`，前面的部分即为**摘要**。其它的一些常用Front-matter设置如下：

```
---
layout: post        # 选择文章页面类型，post(普通文章，可以显示相关文章)，docs(文档，不可显示相关文章)，list(列表)
date:                          # 创建日期
updated:                       # 更新日期
title: 取个什么标题好呢           # 页面标题
seo_title: 关于                 # 网页标题

tags:Hexo                      #单个标签
tags:                          # 设置多个标签        
 - volantis
 - Hexo

pin: true                      # 置顶在首页
archive: false                 # 文章不归档
author: Jon         # 设置文章作者，其他作者信息需要写在数据文件/source/_data/author.yml

top_meta: false   # 是否显示meta标签（文章顶部和底部的日期、分类、更新日期、标签、分享等）
bottom_meta: false # 如果一个页面没有 title 则不会显示 top_meta ，像404、关于页面就可以完全隐藏

thumbnail: https://img.vim-cn.com/17/0c7b02722686d1527a1df807dae0794d995860.png   # 标题右边显示缩略图, 缩略图仅在文章列表和文章页面显示，不会在归档页面显示
cover: true         # 显示文章封面
# 标题右边显示图标
icons: [fas fa-fire red, fas fa-star green]   可以通过 red / blue / green / yellow / orange / theme / accent 来设置图标的颜色

comments: false  # 关闭评论

# 侧边栏显示与否
sidebar: []                   # 不显示
sidebar: [grid, toc, tags]    # 放置任何你想要显示的侧边栏部件
---
```

配置页面插件（说说，渲染公式，Snackbar页面通知等），见[Volantis: 页面配置](https://volantis.js.org/v5/page-settings)。

#### 相关文章插件

`相关文章`功能，需要安装插件 (for layout: post)：

```
$ npm i hexo-related-popular-posts
```

### 数据统计

如果选择[不蒜子](http://busuanzi.ibruce.info/)，取消主题配置文件中的busuanzi注释。

如果选择 [leancloud](https://leancloud.app/) 统计, 你还需前往 leancloud 创建应用并填写下面 leancloud 相关配置。

```
analytics:
  busuanzi: #/libs/busuanzi/js/busuanzi.pure.mini.js #https://cdn.jsdelivr.net/gh/volantis-x/cdn-busuanzi@2.3/js/busuanzi.pure.mini.js
  leancloud: # 请使用自己的 id & key 以防止数据丢失
    app_id: # 应用 APP_ID
    app_key: # 应用 APP_KEY
    custom_api_server: # 国际版一般不需要写，除非自定义了 API Server
```

由于我注册 [leancloud](https://leancloud.app/) 的SMS verification出了问题，所以最后选择了[不蒜子](http://busuanzi.ibruce.info/)。

### 404页面

创建`source/404.md`，写入：

```
---
cover: true
robots: noindex,nofollow
sitemap: false
seo_title: 404 Not Found
bottom_meta: false
sidebar: []
---

{% p logo center huge, 404 %}
{% p center bold, 很抱歉，您访问的页面不存在 %}
{% p center small, 可能是输入地址有误或该地址已被删除 %}
```

------

## 参考资料

[1] Volantis Team, [页面配置](https://volantis.js.org/v5/page-settings), Volantis.org. （最后一次获取时间：2022-03-25）

[2] Volantis Team, [站点配置](https://volantis.js.org/v5/site-settings), Volantis.org. （最后一次获取时间：2022-03-25）

[3] Volantis Team, [Volantis主题源代码的](https://github.com/volantis-x/hexo-theme-volantis/tree/dev)"_config.yml"文件, GitHub. （最后一次获取时间：2022-03-25）

[4] Hexo, [Front-matter](https://hexo.io/zh-cn/docs/front-matter), hexo.io.（最后一次获取时间：2022-03-25）

[5] Volantis Team, [进阶设定](https://volantis.js.org/v5/advanced-settings), Volantis.org. （最后一次获取时间：2022-03-25）

