---
title: Kafka
---

# 概览

## Kafka是什么？

Kafka是一款分布式消息发布订阅系统。高性能、高吞吐量。

内置分区、实现集群、容错、数据复制能力





## Kafka的应用场景

- 日志收集
- 行为跟踪（结合Zipkin，实现链路追踪）



# 安装

 ```shell
wget http://mirror.bit.edu.cn/apache/kafka/2.2.0/kafka_2.11-2.2.0.tgz
tar -zxvf kafka_2.11-2.2.0.tgz
cd kafka_2.11-2.2.0
mkdir log
cd kafka_2.11-2.2.0/config
# 配置server.properties中的log路径和zookeeper路径
vim server.properties
cd ~/Documents/software/kafka_2.11-2.2.0
# 启动。Kafka默认占用9092端口
sh kafka-server-start.sh ../config/server.properties &
 ```



# 应用





# 原理

