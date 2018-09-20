---
title: submodule学习
date: 2018-09-20
---

# submodule常用命令

1. 1、新增：在当前库中没有submodule子模块，可以通过 `git submodule add remote` 添加子模块
2. 2、删除：

```
git rm -fr submodule
删除`./git/config`文件中的相关submodule
删除`./git/submodule`中相关的子模块
```

3. 3、新环境中拉取有子模块的库：
```
初始化`git submodule init`
更新`git submodule update`
```
[相关网页](https://blog.csdn.net/Yan_Chou/article/details/73730793)