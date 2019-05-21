---
title: ShedLock
tags: 
- ShedLock
- 开源项目
- 锁
categories:
- 开源项目
- 锁
---

# ShedLock是什么？

ShedLock是一款基于Java的开源项目，发布在github上：[https://github.com/lukas-krecan/ShedLock](https://github.com/lukas-krecan/ShedLock)

ShedLock是为定时任务设计的一款锁。保证分布式环境下，同一时间同一任务只执行一次。

一个任务只要在某一个节点上运行了，那么它就会获取一个锁，从而防止其他节点或线程上执行同一任务。

> 请注意：只要一个任务在某个节点上运行了，那么其他节点或线程中的同一任务将会跳过执行，而不是等待阻塞。



ShedLock支持多种外部存储形式来进行分布式协调。比如redis、mongo、jdbc数据库、zookeeper等。



总结：ShedLock是对分布式锁的一次封装，让我们在使用锁的过程中更加方便。不用自己实现。



# SpringBoot结合ShedLock

## 引入依赖

```xml
<dependency>
  <groupId>net.javacrumbs.shedlock</groupId>
  <artifactId>shedlock-spring</artifactId>
  <version>2.5.0</version>
</dependency>
<dependency>
  <groupId>net.javacrumbs.shedlock</groupId>
  <artifactId>shedlock-provider-redis-spring</artifactId>
  <version>2.5.0</version>
</dependency>
```



## 引入定时和ShedLock

```java
@Slf4j
@SpringBootApplication
// 允许定时任务
@EnableScheduling
// 允许ShedLock。设置锁最大持有时间是10分钟。
@EnableSchedulerLock(defaultLockAtMostFor = "PT10M")
public class ShedLockApplication {
  public static void main(String[] args) {
    SpringApplication.run(ShedLockApplication.class, args);
    log.info("ShedLockApplication start success!");
  }
}
```



## ShedLock配置Redis锁生产者

shedLock可以有其他的外部存储形式，本案例采用redis

```java
@Configuration
public class ScheduleLockConfiguration {
  @Bean
  public LockProvider lockProvider(RedisConnectionFactory connectionFactory) {
    return new RedisLockProvider(connectionFactory);
  }
}
```



## 添加任务

```java
@Slf4j
@Component
public class DemoTask {
  /**
   * 定时每天凌晨1点执行一次"0 0 1 1/1 * ? "
   * SchedulerLock注解中的name定义了锁的名字，需要保证全局唯一
   */
  @Scheduled(cron = "0 0 1 1/1 * ? ")
  @SchedulerLock(name = "reSendMessageTask")
  public void reSendMessageTask() {
    log.info("定时任务");
  }
}

```

以下是Redis中的锁：

![](https://blog.airaccoon.cn/img/bed/20190521/1558422058022.png)

![](https://blog.airaccoon.cn/img/bed/20190521/1558422086412.png)



### @SchedulerLock

@SchedulerLock的参数有：

- name：定义了锁的名字，需要保证全局唯一
- lockAtMostFor：锁的过期时间，默认是-1。
  - 未设置。任务结束或抛出异常时释放锁。
  - 已设置。任务结束或抛出异常时释放锁。如果任务还没结束，但过了设置的时间，锁也会释放，如果此时其他节点存在同一任务，就可能会造成无法预测的问题。
  - 因此该数据一定要设置成大于一次任务执行的总时间。或者所有的节点的同一任务时间一致，且最好不要存在多次执行的情况。
- lockAtLeastFor
  - 为了防止集群启动的先后或者各节点没有做时钟同步，从而重复起任务的状况。
  - 比如1分钟定时任务执行一次，而least设置是2分钟。那么也就是2分钟是持有锁的最小时间，2分钟后才释放。所以1分钟执行完的定时任务，必须等到2分钟结束，锁释放后，才能再次执行。
- lockAtMostForString
  - PT10M表示超时时间10分钟
  - PT10S表示超时时间10秒
- lockAtLeastForString



# 环境要求

ShedLock依赖于以下环境或依赖包

- JDK8
- slf4j-api
- Spring Framework （可选）