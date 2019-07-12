---
title: Mongo
tags: NoSQL
categories: 分布式
date: 2019-04-25 17:11:23
---



# 概览

是什么？

- 是一款JSON存储的文档型NoSQL数据库



# 连接数据库

```mongo
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

- **mongodb://** 这是固定的格式，必须要指定
- **username:password@** 可选项，如果设置，在连接数据库服务器之后，驱动都会尝试登陆这个数据库
- **host1** 必须的指定至少一个host, host1 是这个URI唯一要填写的。它指定了要连接服务器的地址。如果要连接复制集，请指定多个主机地址。
- **portX** 可选的指定端口，如果不填，默认为27017
- **/database** 如果指定username:password@，连接并验证登陆指定数据库。若不指定，默认打开 test 数据库。
- **?options** 是连接选项。如果不使用/database，则前面需要加上/。所有连接选项都是键值对name=value，键值对之间通过&或;（分号）隔开



# 查询数据库

```mongo
show dbs
```



# 创建数据库

```mongo
use test
```



# 插入数据

```mongo
db.test.insert({'name': '张三'});
```



# 更新数据

```sql

```



# 查询数据

```mongo
db.test.find({'name': '张三'});
-- 结果：
{ "_id" : ObjectId("5caab0b8ac339885f12ba516"), "name" : "张三" }
```



## 分页查询

```mongo
db.test.find().sort({"_id": 1}).skip(10).limit(10);
```





# 创建集合

在 MongoDB 中，不需要创建集合。当插入一些文档时，MongoDB 会自动创建集合。

```shell
db.createCollection('order');
```



# 创建索引

```shell
db.order.ensureIndex({"orderId":1});
```



# 删除集合

类似删除表

```mongo
db.test.drop()
```



# 删除文档

类似删除表行记录

```mongo
db.test.remove({'title':'MongoDB 教程'});
```



# 退出mongo

```mongo
quit();
```

