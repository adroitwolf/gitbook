---
title: spring boot集成swagger2
date: 2021-02-11 15:50:58
tags:
- java
- spring
- spring boot
categories:
- springboot
---

# spring boot集成swagger2


## pom依赖


```xml
<dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.6.1</version>
        </dependency>

        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.6.1</version>
        </dependency>
```

## 创建配置文件


```java

@EnableSwagger2
@Configuration
public class Swagger2Config {

    @Bean
    public Docket CreateRestfulApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("run.app.controller"))
                .paths(PathSelectors.any())
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("ADROITWOLF博客API")
                .description("多用户博客门户网站")
                .termsOfServiceUrl("https://adroitwolf.github.io")
                .version("1.0")
                .build();
    }
}

```

> 里面的basePackage改成自己的controller路径


## 具体使用方法

这里贴一个使用示例：

```java

 @GetMapping("/query")
    @ApiOperation("管理员审核文章列表")
    public BaseResponse getListByExample(){
       xxx                             
    }
```


## 生成API

访问：IP:port/swagger-ui.html