---
title: ElasticSerch
tags:
- ElasticSearch
- 数据库
- 搜索引擎
categaries:
- 数据库
---



本文是为了让小白能够了解ES的基本原理，如为什么搜索这么快（用到了倒排索引）。使用elasticsearch进行创建和搜索文档。



# 一、简单使用

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

- 索引（`_index`）：相当于关系型数据库的数据库
  - 类型（`_type`）：相当于关系型数据库的表（es7.x版本消除了这一概念，不用太关注，默认`_doc`）
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



> Tip1：
>
> 理解这句话：”索引（动词）一个文档到一个索引（名词）中，默认的，一个文档中的每个属性都是能够被索引的（形容词）。“
>
> 索引（动词）：表示插入，类似关系型数据库的`insert`。区别在于当文档已经存在时，新文档会直接替换旧文档而不是无法插入。
>
> 索引（名词）：表示存储文档的地方，类似关系型数据库的数据库
>
> 能被索引的（形容词）：关系型数据库通过在列上增加一个B树索引提高检索速度。ES使用一个“倒排索引”的结构来达到相同的目的。默认的，一个文档中的每个属性都是能够被索引的，是可检索的，没有倒排索引的属性是不能被搜索到的。



> Tip2：
>
> ES中，同一个查询可以使用所有的倒排索引，并以惊人的速度返回结果，而例如mysql只能命中一个列索引。



从上面看出来存储结构中不仅仅包含了文档信息，还包含索引、类型和_id三个元数据。

### 索引（_index）

索引（名词）在此处是一个逻辑上的命名空间。实际上在es中，文档的存储和索引（动词）是在具体的**分片**（后续讲解）上的。一个索引命名空间可以由一个或者多个分片组成。应用程序无需关注这些，这些事情在ES内部做了，对使用者而言是透明的。



### 类型（_type）

类型这一概念在ES 7.x版本之后将被废弃，目前默认`type`统一使用`_doc`。



### _id

ID是字符串，结合`_index`和`_type`就能够唯一定位ES中的一个文档。如果用户不提供，则ES自动创建。



### 创建文档

字段的名字可以是任何合法的字符串，但不可以包含英文句号（.）。

```shell
PUT person/_doc/6
{
  "name": "爱新觉罗天真",
  "age": 39,
  "about": "前朝的王爷",
  "hobby": ["逗鸟"]
}
```

```shell
# 结果
{
  "_index" : "person",
  "_type" : "_doc",
  "_id" : "6",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 30,
  "_primary_term" : 1
}
```



### 更新文档

ES中的文档是**不可改变**的。如果想要修改文档，则需要重建索引。语句和创建文档一致。插入完成后我们会看到`_version`字段和首次插入时是不一样的，且`result`是`updated`。这都表示文档之前已经存在。

```shell
{
  "_index" : "person",
  "_type" : "_doc",
  "_id" : "6",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 31,
  "_primary_term" : 1
}
```

ES会将删除原来的文档，并新增一个全新的文档。我们不再能够对这个标记删除的文档进行操作，但是它并没有立刻消失。ES会在索引更多文档的时候，在后台清理这些标记删除的文档。



### 完全创建新文档

在末尾加上`_create`表示当前索引操作只允许插入，如果已经存在不要更新文档，直接报错。

```shell
PUT person/_doc/6/_create
{
  "name": "爱新觉罗天真",
  "age": 39,
  "about": "前朝的王爷",
  "hobby": ["逗鸟"]
}
```

```shell
# 结果
{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[6]: version conflict, document already exists (current version [2])",
        "index_uuid": "BC91rHxjRfStAqqpkmmoXg",
        "shard": "0",
        "index": "person"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[6]: version conflict, document already exists (current version [2])",
    "index_uuid": "BC91rHxjRfStAqqpkmmoXg",
    "shard": "0",
    "index": "person"
  },
  "status": 409
}
```



### 删除文档

```shell
DELETE person/_doc/6
```

删除文档正如更新文档中提到的，并不会积极从磁盘中删除，而是暂时将文档标记为已删除状态，随着不断索引更多的数据，ES将会在后台清理标记为已删除的文档。



### 并发更新文档造成的数据丢失问题

并发下，用户A更新文档会获取原始文档并创建副本，用户B也会执行相同操作，当用户A更新完成后，是在原始文档基础上的修改，之后会被B的更新覆盖，这就导致A的修改丢失。

很多时候，由于这种问题发生的几率极小，对业务影响不大。比如我们主库使用mysql，只是将mysql数据同步到es，使其可被搜索，这种情况下问题就不是特别大。

但是，当我们使用ES作为主库，比如库存在抢购活动中存在并发问题。如下：

<img src="https://blog.airaccoon.cn/img/bed/20191222/1576994420940.png" width="50%">

变更越频繁，读数据和更新数据的间隙越长，也就越可能丢失变更。

ES不存在mysql悲观锁的机制，他采用的是乐观锁的方式来解决这一数据冲突的问题。原数据在读写过程中，如果发生了修改，那么更新操作将会失败。并由应用程序去判断是否重新读取数据并更新还是直接告知用户重试。

我们使用外部版本号的方式指定下一个版本，如下：

```shell
PUT person/_doc/6?version=3&version_type=external
{
  "name": "爱新觉罗天真",
  "age": 40,
  "about": "前朝的王爷",
  "hobby": ["逗鸟"]
}
```

```shell
# 结果
{
  "_index" : "person",
  "_type" : "_doc",
  "_id" : "6",
  "_version" : 3,
  "_seq_no" : 40,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "爱新觉罗天真",
    "age" : 40,
    "about" : "前朝的王爷",
    "hobby" : [
      "逗鸟"
    ]
  }
}
```

此时，成功更新。我们再次执行相同的更新语句。会抛错`403`，这就是乐观锁版本控制导致的。



### 获取到多个文档

```shell
GET person/_doc/_mget
{
  "ids":[1,2]
}

```





##  搜索

ES赖以生存的就是其强大的搜索能力，文档中的每个字段默认都被索引可查询，且在搜简单索时候所有字段的索引都将被使用，并以惊人的速度返回结果。这是传统数据库做不到的。

ES搜索可以做两件事：

1. 结构化查询。就像传统的关系型数据库一样针对指定字段进行结构化查询；
2. 全文检索。找出所有匹配关键字的文档并按照相关性排序后返回结果。

在全面理解ES搜索之前需要了解三个概念：

1. 映射（Mapping）：描述数据在每个字段内如何存储
2. 分析（Analysis）：全文是如何处理使之可以被搜索的
3. 领域特定查询语言（Query DSL）：Elasticsearch 中强大灵活的查询语言



### 简单搜索

Query-String搜索可以方便快捷的查询，但不支持复杂条件查询。不推荐使用。

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
      "about": "喜欢elastic跳伞"
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



### 分页

ES分页和mysql的分页类似，输入`from`和`size`两个参数就能够获取到一页数据。

- `from`：默认0。偏移量，初始需要跳过的结果数；
- `size`：默认10。一页显示的数量

ES是分布式的，数据存储在多个分片中，分页搜索会对每个分片进行请求。

> Tips：分布式系统中的深度分页问题
>
> 假设搜索从1到10的数据，ES存在5个分片，那么5个分片都分别查询出10条记录，总共50条记录。然后通过协调节点重新排序取出10条记录返回结果。
>
> 那么如果一个请求从1000页开始取10条记录，那么5个分片就都分别会查询10010条记录，到协调节点的时候就存在50050条记录，然后对其排序，放弃50040条记录，返回10条记录。
>
> 这个时候我们就会发现一个深度搜索引发的性能问题。对结果排序的成本随着分页的深度成指数上升。这就是搜索引擎对任何查询都不要返回超过1000个结果的原因。



# 二、映射和分析

所谓映射就是文档的每一个字段在ES是如何存储的；

所谓分析就是存储的每一个字段是如何保证可被搜索的。

## 精确值+全文

ES中的数据分两类：

1. 精确值属性：日期、ID都是精确值，字符串也可以是精确值；
2. 全文属性：指文本数据，人类可以容易识别的语言书写，比如博客文章内容，一般是非结构化的数据。

**精确查询如SQL，全文检索相关性**。。

针对全文检索，ES首先**分析**文档，之后根据结果创建**倒排索引**。



## 分析器

分析器主要作用就是分词并使标准化可搜索，主要分成三个功能

1. 字符过滤器：将文本中的特殊字符过滤掉，比如html标签、`&`转换成`and`等；
2. 分词器：将文本分割成一个个单词；
3. token过滤器：将单词中的一些无用词过滤掉，比如“的”，“了”等。

当我们索引一个文档的时候，该文档的全文域就会被分析器分析成一个个单词并为其创建倒排索引。

同时，当我们在全文域下搜索文本的时候，分析器采用同样的分析，将搜索文本转换成单词进行匹配。

当我们为文档添加一个字符串属性（field）（域）的时候，es会为我们自动创建一个全文字符串属性，使用标准分析器对其分析。如果我们不想要对其进行分析，而是直接使用精确值，则需要我们手动指定这些属性的映射。

分析器只针对全文属性，不针对精确值属性。



## 映射

es中每个属性都有对应的类型，包含以下基础类型：

- 字符型
- 整型
- 浮点型
- 布尔型
- 日期

如果不指定类型，es会根据文档数据自动匹配其类型。

字符型类型被默认是全文属性，因此所有字符串类型属性都会被全文检索，但是我们有时候需要精确值搜索，因此提供了`index`和`analysis`属性控制字符型映射的。

### index

1. `analyzed`：首先分析字符串，然后索引它。换句话说，以全文索引这个域。以全文检索字段排序会消耗大量的内存。
2. `not_analyzed`： 索引这个域，所以它能够被搜索，但索引的是精确值。不会对它进行分析。
3. `no`：不索引这个域。这个域不会被搜索到。

string 域 index 属性默认是 analyzed 。如果我们想映射这个字段为一个精确值，我们需要设置它为 not_analyzed。

### analyzer

对于 `analyzed` 字符串域，用 `analyzer` 属性指定在搜索和索引时使用的分析器。默认， Elasticsearch 使用 `standard` 分析器， 但你可以指定一个内置的分析器替代它



# 三、索引

ES使用**倒排索引**提高搜索效率。那么什么是倒排索引呢？倒排索引是索引技术的一种，它将信息主体解析出n个单词，并对每个单词构建对应的倒排项，组成倒排索引。

倒排索引的数据结构分为两部分：

1. 单词词典：存储了ES中所有的单词（树）
   1. 单词
   2. 单词：通过分析器分析得到关键词
   3. 单词索引：一颗树。所有单词的索引，为了提高单词匹配效率
2. 倒排列表：存储了文档编号，由倒排项组成的列表。（ 数据压缩+有序链表+跳表）
   1. 倒排项（posting）：记录了每个单词在文档中出现了的文档ID列表、偏移值信息等。倒排列表中的每一条记录称为倒排项。
   2. 倒排文件（inverted file）：物理存储倒排列表的文件



![](https://blog.airaccoon.cn/img/bed/20191225/1577269256233.png)

![](https://blog.airaccoon.cn/img/bed/20191225/1577269591101.png)



在倒排索引中，通过单词索引可以找到单词在单词字典中的位置，进而找到倒排列表，有了倒排列表就可以根据ID找到文档了



单词索引：

![](https://blog.airaccoon.cn/img/bed/20191225/1577270405022.png)

我们可以一句话总结倒排索引就是：我们通过单词找到对应的倒排列表，并通过倒排列表中的倒排项找到对应的文档。这就是倒排索引。

由于每个单词对应的文档数量在动态变化，所以倒排列表的建立和维护都较为复杂，但是在查询的时候由于可以一次得到查询关键词所对应的所有文档，所以效率高于正排表。

为什么叫倒排索引？因为是根据属性值中的关键数据找到对应文档id，最终定位到文档，而一般的正向索引则是根据文档id或者其他的索引项找到对应的文档。

![](https://blog.airaccoon.cn/img/bed/20191226/1577350870546.png)

# 四、集群和分布式

## 集群

所有集群名`cluster.name`相同的节点组成了一个集群，每个节点都分担了请求负载压力。按照在集群中的功能不同，节点有以下几个类型：

1. 主节点（`master`）：主要负责控制整个集群，创建删除索引、决定哪些分片分配到哪些节点、跟踪哪些节点是集群一部分。`node.master`设置为true（默认）；
2. 数据节点（`data`）：主要负责文档的增删改查、搜索、聚合。`node.data`设置为true的时候（默认）；
3. 客户端节点（`client`）：`node.master`和`node.data`都是false。表示当前节点仅负责路由请求，从本质上就是一个智能的负载平衡器。
4. 部落节点：配置了`tribe.*`，表示可以连接多个集群，并进行搜索和其他操作。



## 节点发现和选举









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



# 参考文档

[安装过程中的问题以及解决办法](https://blog.csdn.net/happyzxs/article/details/89156068)

[设置账号密码](https://blog.csdn.net/u011265001/article/details/100084335)