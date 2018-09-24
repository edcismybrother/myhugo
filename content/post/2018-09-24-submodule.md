---
layout:     post 
title: submodule
subtitle: 学习如何使用submodule
date: 2018-09-20
author: "皮智文"
URL: "/2018/09/24/submodule/"
tags: ["学习","git","submodule"]
---

# submodule常用命令

1. 新增：在当前库中没有submodule子模块，可以通过 `git submodule add remote` 添加子模块
2. 删除：
```
git rm -fr submodule
删除 `./git/config` 文件中的相关submodule
删除 `./git/submodule` 中相关的子模块
```

3. 新环境中拉取有子模块的库：
```
初始化`git submodule init`
更新`git submodule update`
```
[相关网页](https://blog.csdn.net/Yan_Chou/article/details/73730793)