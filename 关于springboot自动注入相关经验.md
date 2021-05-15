---
title: 关于springboot自动注入相关经验
date: 2021-02-24 14:35:12
tags:
- spring boot
categroies:
- spring boot
---

# 关于spring boot 注入相关经验

------

这里是最近重新学习Spring boot关于注入相关的经验博客，特此记录一下：




## JdbcTemplate注入

在学习java web的时候，有使用过一个叫jdbcTemplate的东西。所以下面这个代码极为熟悉：

```java
@Configuration
public class Springconfig {

 @Bean
    public JdbcTemplate jdbcTemplate(){
        return new JdbcTemplate();
    }

}
```

首先要说几点就是，这个@Configuration是必须要写的，如果没有写，这个Bean是不生效的。

我们创建了一个jdbcTemplate的单例，接下来是怎么注入的问题。


我一开始写的代码是这样的。


```java
class xxx{
    @Autowired
    JdbcTemplate jdbc;
}

```

这里其实是错误的，因为他会自动注入Springboot里面的这个类，我问了一下我的师傅，他和我说：


这是因为xxx这个类没有托管给Spring工厂，加一个@Component就OK了


## 多数据源注入

我最近弄cdc的时候需要用到多数据源，下面是注入时的一些小经验

```yaml
spring:
  datasource:
    dynamic:
      primary: target
      batch:
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/springbatch?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
        username: root
        password: 123
      origin:
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&useSSL=false
        username: root
        password: 123
      target:
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://localhost:3306/target_test?serverTimezone=GMT%2B8&useUnicode=true&characterEncoding=utf8&useSSL=false
        username: root
        password: 123

```


创建一个config

```java

@Configuration
public class DataSourceConfig {

    @Bean("batchDataSource")
    @ConfigurationProperties("spring.datasource.dynamic.batch")
    public DataSource batchDataSource(){
        return DataSourceBuilder.create().build();
    }

    @Bean("originSource")
    @ConfigurationProperties("spring.datasource.dynamic.origin")
    public DataSource originDataSource(){
        xxx
    }


    @Bean("targetSource")
    @ConfigurationProperties("spring.datasource.dynamic.target")
    @Primary
    public DataSource targetDataSource(){
       xxx
    }



}
```

> 说个重要的，当你的Bean后面写了名称的时候，如果想注入这个东西，你需要用到下面的代码


```java
// 第一种
@Autowired
@@Qualifier("batchDataSource") DataSource dataSource;


//第二种

method(@Qualifier("batchDataSource") DataSource dataSource){

}
```
总之@Qualifier这个注解一定要会
