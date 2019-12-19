---
title: ElasticSerch
tags:
- ElasticSearch
- 数据库
- 搜索引擎
categaries:
- 数据库
---

本文档当前目的是为了让小白能够理解和使用elasticsearch。

# 一、基础

## 在ES之前

在ES之前，已经存在了一款优秀的、高性能的全文搜索引擎库Lucene。但Lucene存在以下缺点：

- 只是一个库
- 必须使用Java将Lucene嵌入到应用程序中
- 原理复杂，使用门槛高

基于这些问题，ES应运而生。ES基于Lucene做索引和检索，继承了Lucene的高性能，且对用户透明，开箱即用；提供简单的RESTful API对外暴露，低耦合；同时ES不仅仅是搜索引擎库，还是

- 分布式实时文档存储系统
- 分布式实时搜索引擎系统
- 胜任上百节点扩展，PB级的结构化和非结构化数据

ES能够支持横向扩展至数百甚至数千的节点，处理PB级数据。



> Tips
>
> 关于端口9200和9300
>
> 9300：集群中的每个节点之间的通讯使用了9300，采用TCP传输协议；
>
> 9200：用于外部API通讯，采用Http协议



## 面向文档

ES是面向文档存储的，使用JSON作为序列化格式。存储结构如下：

- 索引：相当于关系型数据库的数据库
  - 类型：相当于关系型数据库的表（es7.x版本消除了这一概念，不用太关注，默认_doc）
    - 文档：相当于关系型数据库的行数据

比如我们索引了一个文档在`person`索引中，该用户文档有名称，生日等属性。（1表示指定索引id）

```shell
## 索引一条文档
PUT /person/_doc/1
{
  "name" : "张三",
  "age" : 15,
  "created" : "2019-01-01 00:00:00",
  "updated" : "2019-01-01 00:00:00"
}
```

```shell
## 检索之前索引的文档
GET /user/_doc/1
```



> Tips
>
> 理解这句话：”索引（动词）一个文档到一个索引（名词）中，默认的，一个文档中的每个属性都是能够被索引的（形容词）。“
>
> 索引（动词）：表示插入，类似关系型数据库的`insert`。区别在于当文档已经存在时，新文档会直接替换旧文档而不是无法插入。
>
> 索引（名词）：表示存储文档的地方，类似关系型数据库的数据库
>
> 能被索引的（形容词）：关系型数据库通过在列上增加一个B树索引提高检索速度。ES使用一个“倒排索引”的结构来达到相同的目的。默认的，一个文档中的每个属性都是能够被索引的，是可检索的，没有倒排索引的属性是不能被搜索到的。



##  搜索

### 简单搜索

Query-String搜索可以方便快捷的查询，但不支持复杂条件查询。

query-string参数以?q=key:value的形式表示

```shell
# 查询person索引下所有文档。默认10条分业
GET person/_search

# 查询person下名字叫张三的数据。查询字符串q=
GET person/_search?q=name:张三
```



### 复杂搜索

ES提供了一个丰富灵活的查询语言叫做**”查询表达式“**，它支持构建更加复杂和健壮的查询。

和之前相同的查询使用查询表达式如下，得到的结果相同：

```shell
GET person/_search
{
  "query": {
    "match": {
      "name": "张三"
    }
  }
}
```



组合条件查询

```shell
GET person/_search
{
  "query": {
    "bool": {
      "must": {
          "match": {"name": "四"}
      },
      "filter": {
        "range": {
          "age": {"gt": 17}
        }
      }  
    }
  }
}
```



### 全文搜索

传统数据库很难做到相关性的全文搜索，而ES却能够轻而易举的做到。比如以下案例，搜索“喜欢跳伞”，就能够将所有包含这四个字的文档查询出来，同时按照**相关性**的大小进行排序。`_score`即相关性得分。

```shell
GET person/_search
{
  "query": {
    "match": {
      "about": "喜欢跳伞"
    }
  }
}
```

![](https://blog.airaccoon.cn/img/bed/20191219/1576737309966.png)



### 短语搜索

上述的所有搜索都只是模糊搜索，想要根据指定的不可分割的短语匹配，类似mysql的like，就需要`match_phrase`的查询。

```shell
GET person/_search
{
  "query": {
    "match_phrase": {
      "about": "喜欢跳伞"
    }
  }
}
```

![](https://blog.airaccoon.cn/img/bed/20191219/1576742220193.png)



### 聚合搜索

ES的聚合搜索类似mysql的group。关键词是`aggs`。比如统计用户的兴趣爱好的分布情况。

```shell
GET person/_search
{
  "aggs": {
    "all_hobbies": {
      "terms": {
        "field": "hobby.keyword",
        "size": 40
      }
    }
  }
}
```

![](https://blog.airaccoon.cn/img/bed/20191219/1576748227783.png)



# 二、何时使用





# 附录：安装

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

# 打开设定es群集名称
cluster.name: my-application
# es当前节点名称，用于区分不同节点
node.name: node-1
# 修改数据目录，此目录为自定义，需要在root用户下创建，且属主属组更改为ela
path.data: /data/es-data
# 日志目录位置，需自己创建，方式同上
path.logs: /var/log/elasticsearch
# 设为false。生产设置为true
bootstrap.memory_lock: false
# 监听访问地址为任意网段
network.host: 0.0.0.0
# 服务监听端口
http.port: 9200
# 设置初始化节点，学习阶段只保留一个节点（之前设置的node.name），否则会启动出错
cluster.initial_master_nodes: ["node-1"]
```

安装完成后设置账号密码：

```shell
cd /usr/local/elasticsearch-7.4.2/
# 生成证书
bin/elasticsearch-certutil cert -out config/elastic-certificates.p12 -pass ""

vim config/elasticsearch.yml
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: elastic-certificates.p12

# 启动elasticsearch
sh bin/elasticsearch

# 使用此命令自动生成几个默认用户和密码
bin/elasticsearch-setup-passwords auto

```



## 参考文档

[安装过程中的问题以及解决办法](https://blog.csdn.net/happyzxs/article/details/89156068)

[设置账号密码](https://blog.csdn.net/u011265001/article/details/100084335)