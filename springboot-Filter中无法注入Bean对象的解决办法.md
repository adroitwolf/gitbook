---
title: springboot Filter中无法注入Bean对象的解决办法
date: 2019-07-24 15:19:18
categories:
- springboot
tags:
- springboot
- 前后端分离
---
# springboot Filter中无法注入Bean对象的解决办法

------------
这次在项目中编写Token代码逻辑的时候，遇到了一个空指针问题，经过排查发现，Filter里面无法利用@Autowired。
所以此次文章用来解决这一问题。


经过查阅资料发现，spring容器初始化Bean的顺序是Listener->Filter->servlet.
那么我可以在Listener里面预先加载我们想要的Bean对象，然后经过Filter构造函数将对象传进去。



下面是具体的代码:

这个是启动监听器。
```java
@Slf4j
@Configuration
@Order(Ordered.HIGHEST_PRECEDENCE)
public class StartLinster implements ApplicationListener<ApplicationEvent> {

    @Autowired
    TokenService tokenService;

    @Override
    public void onApplicationEvent(ApplicationEvent applicationEvent) {
        log.info("提前注入token");
    }
}

```

> 里面TokenService是Service层，也是我们想要注入的对象。


这个是Filter启动的相应代码，可以看到，Listener启动顺序是最高的。
```java
@Bean
   public FilterRegistrationBean<AuthenticationFilter> initAuthenticationFilter(TokenService tokenService){


       FilterRegistrationBean<AuthenticationFilter> authenticationFilterFilter = new FilterRegistrationBean<>();
       AuthenticationFilter authenticationFilter = new AuthenticationFilter(tokenService);

       authenticationFilter.addExcludePatterns("/api/user/login");

       authenticationFilterFilter.setFilter(authenticationFilter);

       authenticationFilterFilter.setOrder(Ordered.LOWEST_PRECEDENCE);

       authenticationFilterFilter.addUrlPatterns("/api/*");

       return authenticationFilterFilter;
   }

```
