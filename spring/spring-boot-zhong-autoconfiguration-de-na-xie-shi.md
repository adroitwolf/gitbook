# Spring Boot的AutoConfiguration

## 什么是AutoConfiguration

在所有的Spring Boot项目中，我们的大部分@Enablexxx，其实不需要手动添加的，在Spring中才需要全部添加，这是因为有

```text
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
```

其中，上面那个包是主要的。下面那个包用来生成元数据，当用户写配置文件的时候会有相应的提示。

## AutoConfiguration的一些具体用法

### 配置多数据源

当我们使用下面这条注解的时候，会自动放弃Spring中自动配置datasource的选项，从而达到自己配置的效果。\(但是这样做并不是很好，多数据源我们可以直接用AOP去做\)

```java
@SpringBootApplication(exclude= {DataSourceAutoConfiguration.class})
```

#### 自己编写starter组件





