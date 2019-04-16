---
title: Mybatis
date: 2019-04-16 14:29:56
tags: ORM
---



# 第一部分：Mybatis是什么

## 一、是什么
1. 是什么：mybatis是一个持久层框架。
2. 特点：支持SQL定制化、支持存储过程和支持高级映射。        
3. 优势：几乎消除了所有的JDBC代码、参数的无脑手动配置和结果集的处理。      
4. 怎么做到：mybatis可以使用简单的xml或者注解来进行配置和映射原始信息，将接口和Java的普通java对象（POJO）映射到数据库记录。      

> tips：面试的时候就可以直接回答这三句话了。这是官方回答啊，绝对不会错。完美。

[Mybatis官网](http://www.mybatis.org/mybatis-3/)

## 二、使用方式
1. 编程式
    - 单独使用Mybatis开发，不集成到任何容器
2. 集成式
    - 比如集成到spring，交由容器管理。

## 三、几个重要的组件
1. Interceptor（plugins）
    - SQL拦截并处理：用户通过实现Interceptor接口可以进行SQL拦截，并作出自定义处理。比如做数据库的读写分离，可以根据这个插件做拦截处理，添加mycat需要的SQL前缀。
    - 比如`select * from t`。通过此拦截器将SQL修改为了`/*!mycat:db_type=master*/select * from t`

2. TypeHandler
    - 类型处理器：java和表字段的类型的转换。可以定制化自己的类型处理器，无论是读取还是存库动作，都可以通过此自定义类型处理机修改对应数据。
    - 比如保存了1，然后自己写了个Integer类型的处理器，并且在要插入的数据后面添加10，那么最终存库的数据是11.

3. environment
    - 保存了数据源信息

## 四、配置文件
### mapper.xml文件解读
```
-- 1. namespace：关联到接口方法，区分类似于类的package
<mapper namespace="net.aitrees.dao.base.UserBaseDao">
</mapper>
-- 2. resultMap/resultType
-- 通过resultMap获取想要的数据。通过resultMap中的association映射java类中的引用属性，并定义前缀别名和java中的属性进行匹配。
<resultMap id="UserCompanyInfoResponseModelMap" type="com.offer.model.account.UserCompanyInfoResponseModel">
    <association columnPrefix="ec_" property="company" javaType="com.offer.entity.base.EjetCompanyEntity">
        <result column="company_id" jdbcType="INTEGER" property="companyId" />
        <result column="user_id" jdbcType="INTEGER" property="userId" />
        <result column="country" jdbcType="VARCHAR" property="country" />
        <result column="company_name" jdbcType="VARCHAR" property="companyName" />
        <result column="create_time" jdbcType="TIMESTAMP" property="createTime" />
        <result column="update_time" jdbcType="TIMESTAMP" property="updateTime" />
    </association>
</resultMap>
<select id="getUserCreatedCompanyInfo" parameterType="java.util.Map" resultMap="UserCompanyInfoResponseModelMap">
    select 
        ec.company_id as ec_company_id, ec.user_id as ec_user_id, ec.country as ec_country, ec.company_name as ec_company_name
    from ejet_company ec
</select>
```
#### 1. namespace
关联到接口方法，区分类似于类的package。
#### 2. resultMap/resultType
1. resultMap：在配置文件中配置的Map，可以定义别名，过滤不需要的字段，可以声明自定义的typeHandler处理。
    - association：关联引用属性的内部属性和数据库的映射关系。可以定义前缀别名和java中的属性进行匹配和父级属性做区别。
2. resultType：将数据库数据直接映射到数据类型的结果集

#### 3. sql

#### 4. CRUD: select\inset\update\delete

#### 5. 动态SQL
foreach、if、when、choose、otherwise等标签

## 五、缓存
### 一级缓存
#### 是什么？
什么是一级缓存？        
回答：一级缓存是基于session的，是数据库级别的。一次会话，多次调用会命中缓存。默认开启。

为什么使用一级缓存？        
回答：减少数据库压力。

一级缓存有什么问题？        
回答：在一个方法里面做两次相同的查询，在其间只要统一数据发生了修改，那么之后拿到的就是脏数据。        
那么为什么还要这么设计？因为这么写代码的概率基本上为0。且这样一种设计的优点>缺点      

### 二级缓存
#### 是什么？
什么是二级缓存？        

回答：将namespace下的所有查询操作结果存入缓存中。其他namespace下针对同一个表数据的操作，当前namespace下的缓存不会被更新。默认是关闭的，不推荐使用。一般使用redis等缓存中间件替代。是基于namespace的。通过cache标签开启。

二级缓存有什么问题？    
1. 脏数据：整个namespace下在第二次查询中从内存中查询，数据库的修改操作不会影响到namespace，出现脏数据的概率非常大。
2. 失效问题：更新策略当基于二级缓存的前提下，进行修改操作，他会把所有缓存清掉，这就违背了缓存存在的意义。

## 六、扩展
### 分页
1. 逻辑分页
    - mybatis底层实现了逻辑分页。是基于已经获取的ResultSet进行分页。即已经返回的数据。因此不推荐可用。DefaultResultSetHandler#handleRowValuesForSimpleResultMap
2. 物理分页
    - select * from user limit 0, 10;
    - 分页插件
        - PageHandler

### 批量操作batch

| 方式                        | 性能             | 限制                        |
| --------------------------- | ---------------- | --------------------------- |
| java的for循环。一个一个插入 | 低。每次都要IO   | /                           |
| foreach拼接SQL              | 性能最高推荐使用 | 有SQL长度限制。定好List大小 |
| Executor.BATCH              |                  | 使用见代码                  |

### 联合查询
嵌套查询：关键标签是`<association><collection>`



# 第二部分：源码分析
## 一句话概括mybatis实现原理
mybatis中mapper的实现原理主要分成两个阶段：解析注册阶段和执行代理阶段。

1. 解析注册阶段：用户通过SqlSessionFactoryBuilder或者SqlSessionFactoryBean创建一个DefaultSessionFactory。在这一过程中，mybatis会通过XmlConfigBuilder解析所有的Xml配置信息并存入Configuration对象中，利用XmlMapperBuilder解析mapper.xml到Configuration对象中。然后通过configuration将mapper接口信息和对应的MapperProxyFactory（Mapper代理工厂）的映射关系注册到MapperRegistry中。
2. 执行代理阶段：用户通过DefaultSessionFactory获取到SqlSession对象，然后调用SqlSession.getMapper方法获取对应的Mapper，最终会通过已经注册的mapper代理工厂创建对应的代理类。然后用户根据这个代理类是执行接口方法，最终会执行代理类MapperProxy的invoke方法。在这个方法中主要做了两件事情：1. 从configuration中获取对应方法的SQL；2. 使用sqlSession执行具体的SQL，SqlSession最终会使用Executor来执行具体的SQL，并将结果返回。

[Mybatis源码分析时序图](https://www.processon.com/view/5c6a1bb4e4b07fada4e8eb35)

## 项目模块
1. mybatis：核心模块
2. mybatis-spring：spring集成模块

## 对比JDBC、Spring JDBC、MyBatis
1. 对比JDBC
    - 几乎消除了所有的JDBC代码，解析结果集的过程，数据映射的过程。只需要编写SQL、配置映射，获得结果。

## 几个重要的类
### SqlSessionFactoryBuilder
使用编程式的Mybatis，则使用此类创建SqlSessionFactory。
### SqlSessionFactory
通过这个类创建SqlSession
### SqlSession
通过SqlSession完成应用和数据库的会话。每完成一次request和response就完成了一次会话。他的生命周期（scope）是method级别的。完成之后就需要关闭。

| 类型              | 作用域（Scope） |
| ----------------- | --------------- |
| SqlSessionFactory | application级别 |
| SqlSession        | session会话级别 |
| Mapper            | method级别      |

# 第三部分：集成Spring分析
mybatis集成spring的实现原理主要是mybatis对spring提供的BeanFactoryRegistryPostProcessor和FactoryBean的实现，从而将SqlSession和Mapper信息注入到spring中。

整个过程是这样的。配置阶段：spring项目中配置SqlSessionFactoryBean和MapperScannerConfigurer这两个bean，分别实现了FactoryBean和 BeanFactoryRegistryPostProcessor接口。SqlSessionFactoryBean用于配置数据源和mapperLocation这个mapper.xml资源路径。这个Bean会将配置的信息存入Configuration对象中，然后返回一个SqlSessionFactory对象。然后是MapperScannerConfigurer对象，用于扫描配置的dao层接口，解析成BeanDefinition对象，然后将接口信息注册到spring的ioc容器中，此时的BeanDefinition的className仍然是Mapper接口，因此是无法进行依赖注入的时候是无法实例化的，为了解决这个问题，mybatis在spring注册beanDefinition之后将对应的className修改成了MapperFactoryBean对象，这个对象持有SqlSessionTemplate对象，然后通过spring的依赖注入，获取对应的mapper的代理类。然后通过SqlSessionTemplate获取SqlSession（此处的SqlSession对象是SqlSessionTemplate，SqlSessionTemplate持有SqlSessionFactory、sqlSessionProxy。），然后根据SqlSession获取Mapper代理对象，过程是这个sqlSessionTemplate先获取持有的sqlSessionProxy（这是一个实现了SqlSession的代理类），通过这个sqlSessionProxy的invoke方法会创建一个DefaultSqlSession，而这个创建的session对象才是真正执行的类。

## 几个重要的类
### SqlSessionFactoryBean
mybatis-spring集成模块的类。使用spring集成式的Mybatis，则使用SqlSessionFactoryBean来创建SqlSessionFactory。内部实际上最终还是调用了SqlSessionFactoryBuilder.     
其中有以下几个属性：
1. plugins：`Interceptor[] plugins`用于注册SQL拦截插件的集合
2. mapperLocations：`Resource[] mapperLocations`配置Mapper.xml的路径
3. dataSource：配置数据源
4. typeHandlers：配置类型处理器



# 第四部分：扩展

主要的扩展：

- mybatis plus
- tk mybatis



# 第五部分：问题

1. mapper在spring中是单例的。但是mapper在mybatis的定义中是方法级别的。为什么可以是单例，即scope是application？
    - 因为对象交由spring管理了。mapper的作用就是CRUD操作。是功能决定的。
2. mybatis在spring集成中没有mapper.xml会不会报错？
    - 不会报错。源码中解析的时候catch住了。但是业务调用的时候肯定会报错。
3. 多个plugin，执行顺序是怎么的？
    - 执行顺序由addPlugins的添加顺序执行的。内部使用for循环调用了plugin执行。
4. 为什么Mapper.java是接口没有实现类？只是使用了xml配置文件就能够运行了？是怎么实现映射的？
    - 在初始化过程中，mybatis会把配置信息包含（mapper.xml）读取到configuration对象中，同时会把所有的namespace对应的Mapper接口信息以及相关的MappperProxyFactory注册到MapperRegistry中。
    - 用户需要获取某一个Mapper执行某一个SQL的时候。就会从这个注册好的Mapper代理工厂中创建一个mapper代理类对象。该对象实现了InvocationHandler，利用JDK的动态代理，最终执行的是这个mapper代理对象的invoke方法。在这个invoke方法中，做了两件事
        - 1. 从configuration中获取对应方法的SQL
        - 2. 通过SqlSession执行对应的SQL。而SqlSession最终是通过Executor对象执行的。
5. 为什么设计者使用Mapper接口而不是实现类？这样的设计有什么好处？
    - 面向接口编程。不管对不对，就是这种思想。便于扩展
    - 从使用层面上解决，如果定义使用类，就会有很多空方法，这样可能会对没有使用过mybatis的人不友好。
    - 如果使用实现类，在代理过程中会调用到这个类的空方法而不是从配置中获取sql进行执行。
6. mybatis的plugin的实现机制是怎样的？
7. 一级缓存和二级缓存的实现机制是怎样的？
    - 一级缓存是Executor对象中的一个localCache对象存储所有执行的结果，在同一个session中查询相同的SQL，除了第一次是查询数据库之外其余都从这个localCache对象中获取。
8. BaseExecutor的localCache是什么用处？
    - 这就是传说中的一级缓存。因为是在BaseExecutor中，作用域跟随BaseExecutor，而BaseExecutor在SqlSession对象中，因此localCache的作用域跟随SqlSession。即一级缓存的作用域（Scope）是session级别的。
9. spring是如何集成mybatis的？