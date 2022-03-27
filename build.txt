#!/bin/bash

# 配置环境
npm install -g hexo-cli
mkdir hexo
cd hexo
hexo init
npm install
rm -rf source/_posts
mkdir -p source/_posts

# 安装主题(你们可以改成自己喜欢的主题)
git clone https://github.com/theme-next/hexo-theme-next themes/next

# 使用新的渲染器
npm un hexo-renderer-marked --save
npm i hexo-renderer-markdown-it --save
# 处理图片
npm i hexo-image-link --save

# 复制文章到指定目录
cd ../
cp -r `ls | grep -v hexo | grep -v config.yml | xargs` hexo/source/_posts
# 复制配置文件
cp config.yml hexo

# 开始构建
cd hexo && hexo g --config config.yml