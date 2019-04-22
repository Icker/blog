---
title: Zookeeper
tags: 分布式协调工具
categories: 分布式
---



# 概览

是什么？

- 分布式协调工具

存储结构

- 键值对

作用

- 服务注册发现中心
- 分布式锁



# 安装

```shell
wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
tar -zxvf zookeeper-3.4.14.tar.gz
cd config
cp zoo_sample.cfg zoo.cfg
cd ../bin
sh zkServer.sh start
```

