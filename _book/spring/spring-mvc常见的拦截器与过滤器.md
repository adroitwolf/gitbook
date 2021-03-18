---
title: spring mvc常见的拦截器与过滤器
date: 2021-02-02 00:20:19
tags:
- spring mvc 
- java
categories:
- java
---

# spring mvc常见的拦截器与过滤器

由于最近面试比较多，所以要临时抱佛脚巩固一下基础知识
-------------------

在聊Filter和handler之前呢，我们要清楚spring mvc的请求流程

> url -> preFilter -> preHandler -> DispatcherServlet -> requestMapping -> postHandler -> afterCompletion -> doFilter


大概是上面的流程，我们也可以用下面的图来表示

[![yejN6K.png](https://s3.ax1x.com/2021/02/01/yejN6K.png)](https://imgchr.com/i/yejN6K)

只要记住 Filter
最开始就要启动就行了



## Filter


这里我只说一下spring mvc里面常见的Filter 


* GenericFilterBean

org.springframework.web.filter.GenericFilterBean.java


```java
public abstract class GenericFilterBean implements
        Filter, BeanNameAware, EnvironmentAware, ServletContextAware, InitializingBean, DisposableBean {

    @Override
    public final void setBeanName(String beanName) {
        this.beanName = beanName;
    }
    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }
    @Override
    public final void setServletContext(ServletContext servletContext) {
        this.servletContext = servletContext;
    }
    @Override
    public void afterPropertiesSet() throws ServletException {
        initFilterBean();
    }
}
//......
```
> 这个类主要实现了spring生命周期的几个接口，方便作为bean纳入IOC容器管理。


* OncePerRequestFilter
spring 里面所有的Filter 都默认继承这个东西，意思是只用一次的Filter


## handler 


HandlerInterceptor

这个接口定义了三个方法，preHandle，postHandle，afterCompletion

* preHandle
这个里头返回false，则会停止继续往下执行
* postHandle
后处理回调方法，实现处理器的后处理，但在渲染视图之前执行，可以在这里额外往视图添加额外的变量等(在preHandle成功执行完，返回true的情况下执行)
* afterCompletion
在preHandle成功执行完，返回true的情况下执行.整个请求处理完毕回调方法，即在视图渲染完毕时回调



# 补充 spring boot过滤器FilterRegistrationBean

由于 Filter写的多了之后，我们要设置一下启动顺序，原来的Filter类实现了OncePreFilter接口之后加个@WebFilter就行了，现在不行了，这个东西不能设置启动顺序，所以需要这个我称之为过滤器的注册中心的东西注册一下


下面，贴出java常见的过滤器注册代码

```java
@Configuration
public class DemoConfiguration {

    @Bean
    public FilterRegistrationBean<Test1Filter> RegistTest1(){
        //通过FilterRegistrationBean实例设置优先级可以生效
        //通过@WebFilter无效
        FilterRegistrationBean<Test1Filter> bean = new FilterRegistrationBean<Test1Filter>();
        bean.setFilter(new Test1Filter());//注册自定义过滤器
        bean.setName("flilter1");//过滤器名称
        bean.addUrlPatterns("/*");//过滤所有路径
        bean.setOrder(1);//优先级，最顶级
        return bean;
    }
}
```