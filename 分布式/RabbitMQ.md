---
title: RabbitMQ
tags: 消息队列
categories: 分布式
---



# RabbitMQ

## 作用

1. 异步
2. 结耦
3. 削峰

## 应用场景

1. 支付系统
2. 分布式事务的最终一致性

## 存在的问题

1. 消息重发
2. 

## MQ中的几个概念

1. 发布-订阅模式
   1. 发布者（Publisher）
   2. 订阅者（Sub）
2. 消息通道（channel）
3. 消息（Message）
4. 消息主机（Broker）
5. 队列（Queue）
6. 交换机（Exchange）
   1. 直连交换机（Direct）
   2. 主题交换机（Topic）
7. 绑定关系（Binding Key）
8. 路由关键字（Routing Key）



交换机通过BindingKey将消息和队列建立绑定关系。



死信队列。没有路由的消息。

1. 消息过期
2. 队列长度到达上限 





## 事务





## 消息消费

消息在什么时候会删除？

1. 发送给消费者的时候就删除了。而不是等待消费者业务处理完成。

