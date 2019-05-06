---
title: git只导出单个文件夹数据
tags: git
categories: 工程化
---



# git只导出单个文件夹数据

git1.7.0以后加入了Sparse Checkout模式，这使得Check Out指定文件或者文件夹成为可能。

```shell
git init
git remote add -f origin http://47.92.96.167:8000/root/blog.git
# 开启sparse checkout 模式
git config core.sparsecheckout true
# 告诉Git哪些文件或者文件夹是你真正想Check Out
# echo 后面第一个是你要check out的文件夹或者文件。>>的右边是.git/info/sparse-checkout不变
echo imgs >> .git/info/sparse-checkout
# 开始从指定分支下拉取想要check out的文件夹或者文件
git pull origin master
```

