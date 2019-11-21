---
title: 规则引擎之easy-rules
tags: 
- 开源项目
- 规则引擎
- easy-rules
categories:
- 开源项目
- 规则引擎
---

# 什么是规则引擎

规则引擎是一种嵌入到应用程序中的组件。实现了将业务决策从程序代码中分离出来，并使用预定义的语义模块编写业务决策。接受数据输入，解释业务规则，并根据业务规则做出业务决策。

> 名词解释：什么是业务规则？
>
> 业务规则包含一组条件和在满足这些条件下执行的操作。
>
> 业务规则在程序中的本质上是一个函数，如Y=f(X1,X2,...,Xn).

规则引擎的好处：将规则从业务代码中抽离出来的，降低了规则迭代成本，增强了代码的维护性和复用性。

让我们想象一下以下场景：

```java
if(number == 1) {
  System.out.println("风险等级A1");
} else if(number == 2) {
  System.out.println("风险等级A2");
} else if(number == 3) {
  System.out.println("风险等级B");
} else if(number == 4) {
  System.out.println("风险等级C");
} else if(number == 5) {
  System.out.println("高风险");
}
//...
```

随着业务条件的增加，`if else`语句越来越多，变得越来越难以维护，这个时候，我们可以使用策略模式来解决这个分支多的问题。

但是，策略模式能解决几个，十几个的问题，但如果策略达到成乾上万种呢？即业务上条件千变万化，需要维护上千个策略类是不切实际的。而规则引擎能够支持表达式语言，可以通过表达式定义多种规则，遇到条件修改的情况，不需要重新编译策略类。而这些就是规则引擎存在的意义。

作为开发人员在使用规则引擎时，通常会存在以下几个步骤：

1. 定义规则对象（条件和执行操作）
2. 注册规则集
3. 定义事实对象
4. 创建规则引擎对象；
5. 向引擎中加载规则集；
6. 向引擎提交需要被规则集处理的事实对象集合；
7. 命令引擎执行;



# easy-rules的使用

以下使用只是基于注解，easy-rules可以基于接口定义规则。同时easy-rules支持MVEL和SpEL表达式来定义规则条件。

[官网](https://github.com/j-easy/easy-rules/wiki)

## 引入maven

```xml
<!-- 6. 规则引擎 -->
<!-- easy rules 核心 -->
<dependency>
  <groupId>org.jeasy</groupId>
  <artifactId>easy-rules-core</artifactId>
  <version>3.3.0</version>
</dependency>
<!-- 继承spring表达式语言进行解析 -->
<dependency>
  <groupId>org.jeasy</groupId>
  <artifactId>easy-rules-spel</artifactId>
  <version>3.3.0</version>
</dependency>
```



## 定义规则

```java
@Rule(name = "购买活动规则")
public class ActivityRule {

  /**
   * 条件
   */
  @Condition
  public boolean when(@Fact("request") ActivityRequest request,
      @Fact("activity") BuyActivity activity) {
    // spEL表达式：#amount>200000&&#terms>=6. 购买金额达到2000元且购买期限>=6个月
    SpELCondition condition = new SpELCondition(activity.getConditions());
    Facts facts = new Facts();
    facts.put("terms", request.getTerms());
    facts.put("amount", request.getAmount());
    // 解析SpEL表达式，满足则返回true
    return condition.evaluate(facts);
  }

  /**
   * 满足条件的执行操作
   */
  @Action
  public void then(@Fact("result") List<BuyActivity> result,
      @Fact("activity") BuyActivity activity) {
    // 将满足条件的活动添加到结果数据中
    result.add(activity);
  }
}

```



## 注册规则集

```java
@AllArgsConstructor
public enum RuleEnums {
  /* 自定义规则枚举 */
  ACTIVITY_RULES("ACTIVITY_RULES", Lists.newArrayList(new ActivityRule()),
      "购买活动枚举");
  @Getter
  private String code;
  @Getter
  private List<Object> ruleList;
  @Getter
  private String desc;
}

// 规则工厂
public class RuleFactory {
  /**
   * 获取规则集
   */
  public static Rules getInstance(RuleEnums enums) {
    // 注册规则集
    Rules rules = new Rules();
    enums.getRuleList().forEach(rules::register);
    return rules;
  }
}

```



## 使用

```java
// 满足条件的相应的活动结果集
List<BuyActivity> result = Lists.newArrayList();
// 所有活动
List<BuyActivity> activities = getFromDB();
// 请求条件
ActivityRequest request = getFromWeb();

activities.forEach(activity -> {
  // 定义事实
  Facts facts = new Facts();
  facts.put("request", request);
  facts.put("result", result);
  facts.put("activity", activity);

  // 获取注册好的规则集
  Rules rules = RuleFactory.getInstance(RuleEnums.ACTIVITY_RULES);

  // 创建规则引擎，加载规则集和事实对象，并执行规则。若满足条件，则执行对应操作。
  new DefaultRulesEngine().fire(rules, facts);
});
```

