---
title: 关于在此文章部署的几行命令
tags:
  - linux-prompt
  - hexo
description: 命令行
cover: 'https://rmimages.gxist.cn/2023/202304171840662.png'
banner: 'https://rmimages.gxist.cn/2023/202304171840662.png'
categories:
  - hexo
abbrlink: 59de1f44
date: 2023-04-12 14:09:58
---
```bash
#在实验室主机上的PATH环境
export PATH="$PATH:/www/server/nodejs/v18.15.0/bin"
hexo clean
hexo generate
hexo deploy
```
