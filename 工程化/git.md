---
title: git
tags: 
- 版本控制
- git
categories: 工程化
---



# git命令补全工具

[https://ohmyz.sh/](https://ohmyz.sh/)



# git常用命令

- `glol`：查看历史记录
- `git pull `: 更新
- `git reset --hard HEAD^`: 还原至上一个提交版本
- `git push`: 发布到远程
- `git branch -aa`: 查看远程分支，并定位到当前分支



## 查看远程信息

```shell
git remote -v
```



## 本地代码提交到远程仓库

```shell
git init
git add .
git commit -m '初始化'
git remote add origin http://IP:PORT/XXX.git
git push -f -u origin master
```



## 新建分支

```shell
# 创建本地分支
git branch branch/project-v0.0.1
# push到远程
git push origin branch/project-v0.0.1
# 拉取远程分支
git checkout /branch/project-v0.0.1
# 设置当前pull、push的默认分支为新增的分支
git branch --set-upstream-to=origin/branch/project-v0.0.1
```



## 删除分支

```shell
# 删除本地分支
git branch -D branch/project-v0.0.1
# 删除远程分支
git push origin --delete branch/project-v0.0.1 
```



## 版本回退

```shell
# 查看日志
git log --pretty=oneline
# 版本回退
git reset --hard b19facd782e859e2d66bbf124d43752db6c2a459
# 强制push到远程（-f）。不加f会失败因为本地已经重置为旧的版本，而远程是新的，导致提交被拒绝。
git push -f
```

![版本回退](https://blog.airaccoon.cn/img/bed/20190705/版本回退.gif)



# 解决每次pull都要输入账号密码的问题

## 第一步

执行以下命令

```shell
git config --global credential.helper store
```

执行完成后，~/.gitconfig文件中会多出来一行

```shell
[credential]
        helper = store
```

## 第二步

执行`git pull`再次输入账号密码，之后就不需要输入密码了。

此时在`/root/.git-credentials`文件中会多出一行

```shell
http://账号名:输入的密码@IP:PORT
```



完成上述两步，就解决了这一问题