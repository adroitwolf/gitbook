# Spring Cloud Alibaba项目构建

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

* config 配置中心 通过搜索文章我发现有 nacos ...config依赖，经过查询发现，这是一个nacos服务中的一个动态配置的东西，相当于咱们的springboot中的application.yml这样一个东西。但是我们知道，修改application不是得重启springboot么，这样太麻烦了，所以config可以动态修改里面的东西。
* discovery 服务注册/发现

总的来说，我们做微服务肯定会有服务注册和发现的，这个就是来做这个的。

### pom文件

并且在父工程中的pom里面写入下面的，如果ide自动生成则无视：

```
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

创建上图中的文件目录，并创建java文件 源代码如下:

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

访问网址并成功返回内容： [![sfm3PU.png](https://s3.ax1x.com/2021/01/20/sfm3PU.png)](https://imgchr.com/i/sfm3PU)

打开nacos管理面板，打开服务列表，如下图所示，则代表注册成功

[![sfmtM9.png](https://s3.ax1x.com/2021/01/20/sfmtM9.png)](https://imgchr.com/i/sfmtM9)

## nacos下的微服务通信

我们在上一篇文章中写了provider模块，并且加载到了nacos的服务列表上面，这次我们开始创建消费者

还是和之前一样创建consumer模块，并且加载pom包，写application项目，创建启动类，这里不再赘述。

### RestTemplate方式

经过查阅nacos文档发现，Http方式有两种请求方式，一种是RestTemplate(说白了就是一个Http请求的API，演示而已，基本项目不适用),另一种是Feign方式(一个假的Http客户端，比较高级，开发中可以使用)，这里先使用RestTemplate方式演示

#### 创建Bean对象

我们先创建一个config文件夹，里面存RestTemplateBean,因为之前我开发SSM的时候，bean都是写在配置文件中的，所以才没有放到实体文件夹中。

```java
/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2021 2021/1/21 10:17
 * Description: nacos消费者配置
 */
@Configuration
public class NacosConsumerConfig {

    /**
    * 功能描述: 相当于 SSM中 <bean id="restTemplate" class="org.springframework.web.client.RestTemplate"/>
    * @Return: org.springframework.web.client.RestTemplate
    * @Author: WHOAMI
    * @Date: 2021/1/21 10:20
     */
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }

}

```

> LoadBalanced 注解是负载均衡的意思

#### 创建Contoller

Bean写好了，接下来就是自动注入。注意我们必须创建一个常量的RestTemplate，我们需要考虑到线程安全，所以Controller代码如下：

```java
/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2021 2021/1/22 10:21
 * Description: 消费者cobtroller
 */
@RestController
public class DemoController {

    private final RestTemplate restTemplate;

    @Autowired
    public DemoController(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }


    @GetMapping(value = "/demo/{string}")
    public String consumerdemoString(@PathVariable("string")String string ){
        return restTemplate.getForObject("http://demo-provider/demo/"+string,String.class);
    }


}
```

> restTemplate.getForObject完成了一次Http请求

启动项目，访问链接，完成!

### Feign方式

我们发现还得写一个Bean,是不是显得很臃肿？我们可不可以写一个接口？Feign提供了解决方案

首先我们要在我们的consumer项目的pom文件中填写Feign的依赖

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

在启动类里面写上 @EnableFeignClients，并且创建一个service文件夹，创建接口服务

代码如下：

```java
/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2021 2021/1/22 10:51
 * Description: Feign方法请求服务
 */
@FeignClient("demo-provider")
public interface DemoService {
    @GetMapping("/demo/{string}")
    String demoString(@PathVariable("string")String string);
}
```

其中FeignClient里面的名字就是服务的名字，接着就像是我们的平常调用service接口那样调用即可。

## nacos的动态配置中心

在上一篇文章中我们提到，nacos实现了可以在外部修改配置，不需要工程重新启动的功能我们现在可以配置一下:

1. 首先，在我们想要编辑的模块下面的配置里面添加这样的依赖

```xml
<dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
```

1. 之后我们需要打开我们的nacos设置界面

[![so0VRf.png](https://s3.ax1x.com/2021/01/22/so0VRf.png)](https://imgchr.com/i/so0VRf)

在这里我们添加配置文件

[![soDF8P.png](https://s3.ax1x.com/2021/01/22/soDF8P.png)](https://imgchr.com/i/soDF8P)

> 名字随便起，但是格式一定是像上面的箭头所指的一致。之后就只需要把我们服务中的配置文件里面的内容复制粘贴下就好了

1. 下一步我们要编写一个文件 " bootstrap.properties"文件

> 我们要知道spring boot项目文件加载顺序是 bootstrap.properties -> bootstrap.yml -> application.properties -> application.yml

所以我们知道有的时候我们明明在application配置文件中写了东西但是没有加载进去，可能就是因为优先级太低没有读取的原因

```properties
spring.application.name=demo-consumer
spring.cloud.nacos.config.server-addr=192.168.3.10:8848
spring.cloud.nacos.config.file-extension=yaml

spring.profiles.active=dev
```

启动，测试访问服务，成功

> 由于我们项目在实际的jar包启动一般都是这样的 java -jar xxx.jar --spring.profile.active=prod/dev 所以我们的配置文件一般不写启动环境设置，而是不同的环境不同的配置文件

### nacos动态刷新

我们可以测试一下他的动态刷新功能，我们刚才的nacos配置文件下面随便填点东西，例：test.name="张巨根"

然后我们在我们想要获取配置的测试controller类下面添加\*\*@RefreshScope\*\*注解，之后通过@Value注解获取配置就可以了。

如果在nacos修改配置之后，页面刷新会动态显示配置
