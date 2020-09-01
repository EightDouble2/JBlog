---
title: "Hugo博客环境搭建"
date: 2020-04-18T14:51:43+08:00
draft: false
tags: [ "Hugo" ]
categories: [ "环境搭建" ]
---
# Hugo博客环境搭建

[hugo官网](https://gohugo.io/)

## 使用homebrew下载安装hugo
`brew install hugo`

## 查看是否安装成功
`hugo version`

## 创建博客项目
`hugo new site JBlog`

## 克隆主题
[hugo主题官网](https://themes.gohugo.io/)
- 进入项目目录：`cd $HUGO_ROOT`
- 克隆主题：`git clone https://github.com/xiaoheiAh/hugo-theme-pure themes/pure`

## 客制化主题
编辑根目录的`config.yml`文件

## 启动hugo服务器
`hugo server -t pure --buildDrafts`  
访问本地服务地址：[http://localhost:1313/](http://localhost:1313/)

## 创建博客
`hugo new post/blog.md`

# 将博客项目推送到GitHub

## 生成静态页面
`hugo --buildDrafts`

## 创建github项目
项目名为`https://eightdouble2.github.io/`

## 推送生成的静态文件
- 进入生成的静态文件目录：`cd public`
- 初始化git：`git init`
- 暂存文件：`git add .`
- 提交文件：`git commit -m "JBlog Init"`
- 与远程仓库建立映射：`git remote add origin https://github.com/EightDouble2/eightdouble2.github.io.git`
- 拉取远程仓库代码：`git pull origin master --allow-unrelated-histories`
- 推送：`git push -u origin master`

访问GitHub博客项目地址：[https://eightdouble2.github.io/](https://eightdouble2.github.io/)