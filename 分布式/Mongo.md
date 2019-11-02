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



# 结合Spring Boot

## 依赖加载

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```



## 参数配置

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://raccoon:123456@localhost:27017/universe
```



## 实体映射

`Spring Data MongoDB`提供了映射注解，我们在实体类上使用这些注解就能够映射成对应的mongo表。

1. `@Id`：标记ID字段。默认是`ObjectId`类，没有标记此注解，则我们无法读取到ID；
2. `@Document`：标记此实体类是mongo集合映射类，`collection`参数指定集合名称；
3. `@Indexed`：为指定字段创建索引。`direction`参数可以指定排序方向，升或降序；
4. `@CompoundIndex`：创建复合索引。`def`参数指定索引包含的字段以及排序方向；
5. `@Transient`：标记为原生字段。不映射到mongo中；
6. `@Field`：指定字段在mongo中的名称；
7. `@DBRef`：指定与其他集合的级联关系，不添加此注释时对象将会保存具体的实体值，而添加了此注释当前集合保存的是关联集合的id



# 安装Mongo

```shell
sudo touch /etc/yum.repos.d/mongodb-org.repo

sudo vi /etc/yum.repos.d/mongodb-org.repo
[mongodb-org]
name=MongoDB Repository 
baseurl=http://mirrors.aliyun.com/mongodb/yum/redhat/7Server/mongodb-org/3.2/x86_64/ 
gpgcheck=0 
enabled=1

vim /etc/mongod.conf
net:
port: 27017
bindIp: 127.0.0.1 // mongodb 默认绑定的IP地址


mongo     // 本地连接数据库
use admin    // 切换到admin数据库，没有会自动添加
db.createUser({	 // 创建管理员用户
 user: "raccoon",
 pwd: "116511",
 roles: [{role: "root", db: "admin"}]	// 角色：超级管理员，数据库：admin
 }
)

# 外网连接

```

