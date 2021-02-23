---
title: Spring Cloud Alibaba项目构建(二)
date: 2021-01-22 10:45:57
tags:
- java
- spring
categories:
- java
---
# Spring Cloud Alibaba项目构建(二)

-----------
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

2. 之后我们需要打开我们的nacos设置界面

[![so0VRf.png](https://s3.ax1x.com/2021/01/22/so0VRf.png)](https://imgchr.com/i/so0VRf)


在这里我们添加配置文件 

[![soDF8P.png](https://s3.ax1x.com/2021/01/22/soDF8P.png)](https://imgchr.com/i/soDF8P)

> 名字随便起，但是格式一定是像上面的箭头所指的一致。之后就只需要把我们服务中的配置文件里面的内容复制粘贴下就好了


3. 下一步我们要编写一个文件 " bootstrap.properties"文件

> 我们要知道spring boot项目文件加载顺序是
bootstrap.properties -> bootstrap.yml -> application.properties -> application.yml

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


然后我们在我们想要获取配置的测试controller类下面添加**@RefreshScope**注解，之后通过@Value注解获取配置就可以了。



如果在nacos修改配置之后，页面刷新会动态显示配置


