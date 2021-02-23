---
title: Spring Cloud Alibaba项目构建(一)
date: 2021-01-08 10:26:09
tags:
- java
- spring
categories:
- java
---

# Spring Cloud Alibaba项目构建(一)

## 版本说明

由于我之前的eblog单机版本的springboot版本为2.1.4,所以之后的所有的项目都以该项目版本为基础。

[![sWb0ld.png](https://s3.ax1x.com/2021/01/20/sWb0ld.png)](https://imgchr.com/i/sWb0ld)


我们打开spring官网里面的spring cloud alibaba的api界面，这里我选择的是2.1.2版本 spring cloud 是GreenwichSR5版本



| Spring Cloud Version        | Spring Cloud Alibaba Version | Spring Boot Version |
| --------------------------- | ---------------------------- | ------------------- |
| Spring Cloud Hoxton.SR3     | 2.2.1.RELEASE                | 2.2.5.RELEASE       |
| Spring Cloud Hoxton.RELEASE | 2.2.0.RELEASE                | 2.2.X.RELEASE       |
| Spring Cloud Greenwich      | 2.1.2.RELEASE                | 2.1.X.RELEASE       |
| Spring Cloud Finchley       | 2.0.2.RELEASE                | 2.0.X.RELEASE       |
| Spring Cloud Edgware        | 1.5.1.RELEASE                | 1.5.X.RELEASE       |




## 项目构建

由于我们搞的是微服务，所以首先要有一个父工程来规定依赖版本

我们首先创建一个maven工程，将packaging设置成pom

下面是我的父工程的pom示例

```xml

<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.cloud</groupId>
    <artifactId>eblog</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>pom</packaging>
    <description>eblog父项目</description>



    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <java.version>1.8</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <spring.cloud.alibaba.version>2.1.0.RELEASE</spring.cloud.alibaba.version>
        <spring.cloud.version>Greenwich.SR5</spring.cloud.version>

    </properties>


    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <dependencyManagement>

        <dependencies>
             <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>${spring.cloud.alibaba.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>

            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            

        </dependencies>
    </dependencyManagement>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>

```


## 与nacos整合

> 建议查看nacos[官方网站](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)springcloud整合的官方文档


> 保证首先将nacos服务打开，如果没有安装请查看博客内相关安装教程

首先我们创建一个名字为**provider**的模块用来注册(其实正常项目中所有的项目大部分都是注册到注册中心的，这里只是为了演示，只写了一个模块)，创建pom文件
> 最终的文件目录为下图：


[![sfew9g.png](https://s3.ax1x.com/2021/01/20/sfew9g.png)](https://imgchr.com/i/sfew9g)


### nacos jar包说明

* config 配置中心
    通过搜索文章我发现有 nacos ...config依赖，经过查询发现，这是一个nacos服务中的一个动态配置的东西，相当于咱们的springboot中的application.yml这样一个东西。但是我们知道，修改application不是得重启springboot么，这样太麻烦了，所以config可以动态修改里面的东西。
* discovery 服务注册/发现

总的来说，我们做微服务肯定会有服务注册和发现的，这个就是来做这个的。




### pom文件

并且在父工程中的pom里面写入下面的，如果ide自动生成则无视：
```pom
<modules>
        <module>provider</module>
</modules>
```


provider文件中的pom为下面所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>eblog</artifactId>
        <groupId>com.cloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>provider</artifactId>

    <dependencies>
        <!--nacos discovery依赖-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>

</project>

```


### 启动类
我们spring boot项目需要程序入口：

创建上图中的文件目录，并创建java文件
源代码如下:

```java
package com.cloud.demo.provider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2021 2021/1/20 20:36
 * Description: 项目启动入口
 */
@SpringBootApplication
@EnableDiscoveryClient
public class ProviderApplication {
    public static void main(String args[]){
        SpringApplication.run(ProviderApplication.class,args);
    }
}

```

* SpringBootApplication 注解是springboot的扫描类
* EnableDiscoveryClient 注解是nacos扫描类

### 控制层

我随便写一个controller的demo

```java

package com.cloud.demo.provider.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2021 2021/1/20 20:42
 * Description: 测试控制层
 */
@RestController

public class DemoContoller {

    @GetMapping(value = "/demo/{string}")
    public String demoTesting(@PathVariable("string") String string){
        return "hello" + string;
    }
}

```

### resources文件

resources文件中的application.yml文件：

```yml
spring:
  application:
    name: demo-provider
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.3.10:8848
        # 这里面的nacos地址是你自己的nacos-server的地址
server:
  port: 8081

# 健康监控
management:
  endpoints:
    web:
      exposure:
        include: '*'



```


### 测试

首先启动这个provider模块项目

访问网址并成功返回内容：
[![sfm3PU.png](https://s3.ax1x.com/2021/01/20/sfm3PU.png)](https://imgchr.com/i/sfm3PU)

打开nacos管理面板，打开服务列表，如下图所示，则代表注册成功

[![sfmtM9.png](https://s3.ax1x.com/2021/01/20/sfmtM9.png)](https://imgchr.com/i/sfmtM9)




