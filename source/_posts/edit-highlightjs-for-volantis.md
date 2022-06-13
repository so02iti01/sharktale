---
title: Hexo|实现代码高亮的多彩——针对 volantis 主题
date: 2022-05-10 22:03:09
updated:
tags:
- volantis
- highlightjs
- hexo
---

volantis 主题的默认代码块是单一颜色的，所以如果需要实现不同代码部分的多种颜色，需要自己进行修改。而如果使用 Hexo 的 highlightjs，进行代码块的 layout 自定义，则不能显示代码行数，进而添加 [highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/) 实现显示代码行数。操作参考 [highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/) 官方文档，只是针对 volantis 主题需要修改的文件有一些小差别。

<!--more-->

## highlightjs-line-numbers.js

[highlightjs](https://highlightjs.readthedocs.io/en/latest/line-numbers.html) 的作者表示代码添加行号很多余，可以把代码分段插入来避免可能的问题。作者同时也表示，“the only way to show that they are better is to set up some usability research on the subject. I doubt anyone would bother to do it”。但是，就是有人 bother to do it.

[highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/) 就是其中一个实现代码行号的 plugin

highlightjs-line-numbers.js 的安装，可以采用 Bower/Npm，也可以直接从 CDN引入。

## 修改highlight.js

修改 highlight.js ，为代码添加行数显示

对于volantis 主题，则需要在`~\themes\volantis\layout\_plugins\highlight\` 的 `script.ejs` 末尾添加

```javascript
<% if (theme.plugins.highlightjs.js) { %>
<%- js(theme.plugins.highlightjs.js) %>
<script src="//cdn.jsdelivr.net/npm/highlightjs-line-numbers.js@2.8.0/dist/highlightjs-line-numbers.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
<script>hljs.initLineNumbersOnLoad({singleLine:true});</script>
<script>
<% } %>
volantis.pjax.push(()=>{
    document.querySelectorAll('pre code').forEach((block) => {
    hljs.highlightBlock(block);
    hljs.lineNumbersBlock(block,{singleLine:true});
    });
},"highlightjs")
</script> 
```

## 创建代码块样式文件

这部分参考了 [GOOPHER](https://goopher.tk/) 的操作

在 `~/themes/volantis/source/css/` 目录下创建 `_other` 文件夹，并在里面创建一个名为 `codeblock.styl` 的文件，文件内容直接复制自 [highlightjs-line-numbers.js](https://github.com/wcoder/highlightjs-line-numbers.js/) 说明的 cool style 部分

```javascript
/* for block of numbers */
.hljs-ln-numbers {
    -webkit-touch-callout: none;
    -webkit-user-select: none;
    -khtml-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;

    text-align: center;
    color: #ccc;
    border-right: 1px solid #CCC;
    vertical-align: top;
    padding-right: 5px;

    /* your custom style here */
}

/* for block of code */
.hljs-ln-code {
    padding-left: 10px;
}
```

打开  `~/themes/volantis/source/css/` 目录下的 `style.styl` 文件，添加以下内容，引入样式

```css
@import '_other/*'
```

## 修改 _config.yml 和 _config.volantis.yml

打开  `_config.yml`  ，设置 disable `highlight` 和 enable `highlightjs`

```javascript
# 修改的内容
highlight:
  enable: false
  hljs: true
```

打开  `_config.volantis.yml`  ，去除#

```
highlightjs:
    copy_code: true
    # 删除下面2行前面的#
    js: ...
    css: ....
```

 `_config.volantis.yml` 里一些可能会用到的修改语句

```
  light:
    # 区块和代码块背景色(填充)
    block: '#f6f6f6'     
    # 代码块高亮时的背景色(边框)
    codeblock: '#FFF7EA'
    # 行内代码颜色
    inlinecode:  '#4282d7'      # '#c74f00'
```

最后的效果![image-script](https://s2.loli.net/2022/05/16/wNBzWqP7JSVuUmp.png)

