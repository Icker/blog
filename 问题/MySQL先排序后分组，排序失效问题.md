---
title: MySQL先排序后分组，排序失效问题
tags: 
- 问题
- mysql
categories:
- 问题
- mysql
---



# 问题描述

由于历史原因，我在今天进行了针对学员跟进记录的线上数据订正，其中需要写一条SQL查询每个学生最小的跟进时间，并或者这个时间更新到需要订正的表中。但发现，我起初写的SQL在实际情况下会存在部分数据不是最小值的情况，导致我订正数据存在问题。立马检查，对原有SQL进行了修改。



## 产生过程以及解决方案

| ID                  | 学生ID student_id   | 跟进时间remind_time |
| ------------------- | ------------------- | ------------------- |
| 113756308207495168  | 1025303812644028512 | 2019-06-24 11:32:00 |
| 1054300615720566880 | 1025303812644028512 | 2019-09-23 17:17:00 |

表中存在以上数据，应该获取上述第一条记录。

此时，我写了以下SQL：

```sql
select student_id, remind_time from (
  select student_id, inst_id, remind_time, progress from student_feedback order by remind_time asc
) as sf
group by sf.student_id;
```

SQL的思路很简单，我先按时间从小到大排序，然后对这个排序后的数据进行分组，理论上分组会从第一条记录上获取值，我想的就是应该就能够获取跟进时间是2019-06-24 11:32:00的记录。

但实际上却获得了跟进时间是2019-09-23 17:17:00的记录。

这个时候，我暂时还不知道为什么会这样。但为了快速解决问题，我做了细微的调整，在remind_time上添加了min方法，得到了以下SQL：

```sql
select student_id, min(remind_time) as remind_time from (
  select student_id, inst_id, remind_time, progress from student_feedback order by remind_time asc
) as sf
group by sf.student_id;
```

这个时候，我才真正的获取到了最小的记录。



## 问题的原因

经过百度，发现网上有很多人遇到了相同的问题，大多数的回答是mysql5.7的优化器对排序的SQL做了新的优化，子查询的排序会被忽略，导致group的数据并不是排序后的数据。5.7之前是没有问题的。

解决办法有两种：

1. 子查询中添加limit
2. 使用聚合函数先获取想要的数据，然后关联原表获取数据。（我用的就是这个方式）