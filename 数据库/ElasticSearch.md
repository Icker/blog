---
title: ElasticSerch
tags:
- ElasticSearch
- 数据库
- 搜索引擎
categaries:
- 数据库
---



# 一、是什么

ElasticSearch是一款开源的分布式搜索引擎。存在以下3个特点：

1. 支持全文检索。单个字段都支持；
2. 支持分布式存储
3. 提供WEB RestFul风格接口进行增删改查数据







# 如何使用



# 一些概念

## 索引：index

es中的索引类似于mysql中的database（schema），必须使用小写命名，是存储es数据的地方。



## 类型：type

是对index的细分，类似于mysql中的表。



## 文档：document

es存储的最基本单元。类似于mysql中的行数据。



## 字段：field

es存储的最小单元。类似于mysql中的列。



# 安装

```shell
[root@raccoon local]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.2-linux-x86_64.tar.gz
[root@raccoon local]# tar -zxvf elasticsearch-7.4.2-linux-x86_64.tar.gz
[root@raccoon local]# groupadd elastic
[root@raccoon local]# useradd -g elastic elastic
[root@raccoon local]# passwd elastic
[root@raccoon local]# vim /etc/sudoers
```

![](https://blog.airaccoon.cn/img/bed/20191129/1575021508366.png)

```shell
[root@raccoon local]# chown -R elastic.elastic /usr/local/elasticsearch-7.4.2/
[root@raccoon local]# vim /usr/local/elasticsearch-7.4.2/config/elasticsearch.yml

//打开设定es群集名称
cluster.name: my-application
//es当前节点名称，用于区分不同节点
node.name: node-1
//修改数据目录，此目录为自定义，需要在root用户下创建，且属主属组更改为ela
path.data: /data/es-data
//日志目录位置，需自己创建，方式同上
path.logs: /var/log/elasticsearch
//elasticsearch官网建议生产环境需要设置bootstrap.memory_lock: true 但亲试没用 得设为false
bootstrap.memory_lock: false
//监听访问地址为任意网段
network.host: 0.0.0.0
//服务监听端口
http.port: 9200
```

