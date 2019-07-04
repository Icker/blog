---
title: whistle
tags: 
- 抓包工具
- whistle
- 代理工具
categories: 
- 抓包工具
- whistle
---



# whistle

[whistle文档](https://github.com/avwo/whistle/blob/master/README-zh_CN.md)

whistle是一款抓包工具、代理工具、调试工具。



![whistle](https://blog.airaccoon.cn/img/bed/20190704/whistle.gif)



## 安装

```shell
npm install  -g  whistle
w2 -h
# 启动，不设置端口默认使用8899
w2 start -p 8899
```



## 作用

1. 将具体地址修改成本地地址，方便调试
2. 作为代理服务器，在手机上设置whistle代理IP，就可以将指定访问转化为本地访问，从而实现抓包调试。