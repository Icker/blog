---
title: 初次使用spring连接kafka遇到的问题
tags: 
- 问题
- kafka
categories:
- 问题
- kafka
---



# 初次使用spring连接kafka遇到的问题

org.apache.kafka.clients.networkclient broker may not be available

初次安装kafka，并通过项目连接，一直提示错误，连接不上kafka。

![](https://blog.airaccoon.cn/img/bed/20190516/1557991794968.png)



上网查后发现是kafka配置没配好，没有对外开放host、port。于是我就做了对应的修改：

![](https://blog.airaccoon.cn/img/bed/20190516/1557991932238.png)



重启kafka，连接成功～

