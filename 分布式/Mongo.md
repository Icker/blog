---
Mongo
---



# 概览

是什么？

- 是一款JSON存储的文档型NoSQL数据库



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



# 查询数据

```mongo
db.test.find({'name': '张三'});
-- 结果：
{ "_id" : ObjectId("5caab0b8ac339885f12ba516"), "name" : "张三" }
```



# 退出mongo

```mongo
quit();
```

