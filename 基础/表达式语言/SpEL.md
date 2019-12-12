---
title: SpEL
tags:
- 表达式语言
- SpEL
categories:
- 表达式语言
- SpEL
---



# 是什么

SpEL是Spring使用的一种表达式语言。类似于EL表达式，他能够在运行时操作对象符号。拥有方法调用（Method invocation）、变量替换（Variables）、比较运算、bean引用（Bean references）等功能。



# 怎么用

## 符号

- `''`（字符串）
- `#`（声明变量符号）

### 四则运算符

- `+`
- `-`
- `*`
- `/`

### 比较运算符

- `lt` (`<`)
- `gt` (`>`)
- `le` (`<=`)
- `ge` (`>=`)
- `eq` (`==`)
- `ne` (`!=`)
- `div` (`/`)
- `mod` (`%`)
- `not` (`!`)



### 逻辑运算符

- `and`
- `or`
- `not`



## 使用

### 文字表达式（Literal expressions）

```java
/**
 * 文字表达式。''表示字符串
 */
private static void literalExpressions() {
  SpelExpressionParser parser = new SpelExpressionParser();
  System.out.println(parser.parseExpression("'你好，张三'").getValue());
  System.out.println(parser.parseExpression("6.0221415E+23").getValue());
  System.out.println(parser.parseExpression("0x7FFFFFFF").getValue());
}
```



### 基于`matches`操作符的正则表达式

```java
/**
 * 基于`matches`操作符的正则表达式
 */
private static void regularExpression() {
  SpelExpressionParser parser = new SpelExpressionParser();
  String regular = "'^\\w+([-+.]\\w+)*@\\w+([-.]\\w+)*\\.\\w+([-.]\\w+)*$'";
  System.out.println(parser.parseExpression("'test@qq.com' matches " + regular).getValue());
  System.out.println(parser.parseExpression("'test@com' matches " + regular).getValue());
}
```



### 四则运算符、比较操作符和逻辑运算符

```java
/**
 * 四则运算符、比较操作符和逻辑运算符
 * 实数默认会使用Double.parseDouble()解析
 */
private static void operator() {
  SpelExpressionParser parser = new SpelExpressionParser();
  System.out.println("4 + 3 = " + parser.parseExpression("4 + 3").getValue());
  System.out.println("4 - 3 = " + parser.parseExpression("4 - 3").getValue());
  System.out.println("4 / 3 = " + parser.parseExpression("4D / 3D").getValue());
  System.out.println("4 * 3 = " + parser.parseExpression("4 * 3").getValue());
  System.out.println("3.0 > 2 = " + parser.parseExpression("3.0 > 2").getValue());
  System.out.println("3.0 gt 2 = " + parser.parseExpression("3.0 gt 2").getValue());
  System.out.println("3.0 < 2 = " + parser.parseExpression("3.0 < 2").getValue());
  System.out.println("3.0 lt 2 = " + parser.parseExpression("3.0 lt 2").getValue());
  System.out.println("'天才' == '天才' = " + parser.parseExpression("'天才' == '天才'").getValue());
  System.out.println("'蠢材' eq '蠢材' = " + parser.parseExpression("'蠢材' eq '蠢材'").getValue());
  System.out.println("999999999999999L == '999999999999999L = " + parser.parseExpression("999999999999999L == 999999999999999L").getValue());
  System.out.println("999999999999999D eq '999999999999999D = " + parser.parseExpression("999999999999999D eq 999999999999999D").getValue());
  System.out.println("3 % 2 = " + parser.parseExpression("3 % 2").getValue());
  System.out.println("3 mod 2 = " + parser.parseExpression("3 mod 2").getValue());
  System.out.println("3 != 2 = " + parser.parseExpression("3 != 2").getValue());
  System.out.println("3 ne 2 = " + parser.parseExpression("3 ne 2").getValue());
  System.out.println("3 >= 2 = " + parser.parseExpression("3 >= 2").getValue());
  System.out.println("3 ge 2 = " + parser.parseExpression("3 ge 2").getValue());
  System.out.println("3 <= 3 = " + parser.parseExpression("3 <= 3").getValue());
  System.out.println("3 le 3 = " + parser.parseExpression("3 le 3").getValue());
  System.out.println("!true = " + parser.parseExpression("!true").getValue());
  System.out.println("3 le 3 or 4 > 5 = " + parser.parseExpression("3 le 3 or 4 > 5").getValue());
  System.out.println("3 le 3 and 4 > 5 = " + parser.parseExpression("3 le 3 and 4 > 5").getValue());
}
```



### 执行方法

```java
/**
 * 执行方法
 */
private static void callMethod() {
  SpelExpressionParser parser = new SpelExpressionParser();
  System.out.println(parser.parseExpression("'张三'.equals('张三')").getValue());
  System.out.println(parser.parseExpression("new StringBuilder('张三').append('是').append('天才').toString()").getValue());
  System.out.println(parser.parseExpression("new StringBuilder('张三').append('是').append('天才').toString()").getValue());
}
```



### 访问对象属性、数组、集合

```java
/**
 * 通过表达式字符串从对象中获取指定的数据
 * 访问对象属性、数组、集合
 */
private static void getValueFromObj() {
  SpelExpressionParser parser = new SpelExpressionParser();
  User user = new User().setCountry("china").setName("张三");
  System.out.println(parser.parseExpression("country").getValue(user));
  System.out.println(parser.parseExpression("country eq 'china'").getValue(user));

  ClassDemo demo = new ClassDemo();
  System.out.println(parser.parseExpression("users.get(0).getName()").getValue(demo));
  System.out.println(parser.parseExpression("users[2].name").getValue(demo));
  System.out.println(parser.parseExpression("country['俄罗斯']").getValue(demo));
}


@Data
@NoArgsConstructor
@AllArgsConstructor
static class ClassDemo {
  private List<User> users = Lists.newArrayList(
      new User().setCountry("中国").setProvince("浙江").setName("鲁迅"),
      new User().setCountry("中国").setProvince("四川").setName("大熊猫"),
      new User().setCountry("希腊").setProvince("雅典").setName("雅典娜")
  );

  private Map<String, String> country = MapUtil.builder()
      .with("中国", "China")
      .with("美国", "USA")
      .with("俄罗斯", "Russia")
      .build();
}
```



### 变量处理



```java
/**
 * 变量处理
 */
private static void variables() {
  SpelExpressionParser parser = new SpelExpressionParser();
  EvaluationContext context = new StandardEvaluationContext();
  context.setVariable("name", "张三");
  context.setVariable("age", 2);
  context.setVariable("gender",  "男");
  
  User user = new User();
  parser.parseExpression("name = #name").getValue(context, user);
  System.out.println(user);

  System.out.println(parser.parseExpression("#age>2").getValue(context));
  System.out.println(parser.parseExpression("#age==2").getValue(context));
  System.out.println(parser.parseExpression("#age==2&&#gender=='男'").getValue(context));
  System.out.println(parser.parseExpression("#age==2&&#gender=='女'").getValue(context));
}

```



[文档](https://docs.spring.io/spring-framework/docs/current/spring-framework-reference/core.html#expressions)