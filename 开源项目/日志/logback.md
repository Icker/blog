---
title: logback
tags: 
- 日志
- logback
categories:
- 日志
- logback
---



# 一、简介和架构

## 1.1 简介

# logback是日志系统，它继承自log4j，比其他的日志系统更快更小。



## 1.2 架构

logback主要构建在三个组件上：Logger、Appender、Layout。

- Logger负责设定日志的输出级别：trace、debug、info、warn、error；默认debug级别
- Appender负责确定日志输出的目的地，比如控制台、文件、消息队列、数据库等。一个logger可以输出到多个Appender中；
- Layout负责确定输出日志的格式，对日志进行格式化。



![](https://blog.airaccoon.cn/img/bed/20200311/1583918668217.png)





# 二、组件



## 2.1 Appenders



## 2.2 Encoder



## 2.3 Layouts



## 2.4 Filters



## 2.5 MDC



## 2.6 Receiver







# 三、

