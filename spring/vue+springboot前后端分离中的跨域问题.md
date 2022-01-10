---
title: vue+springboot前后端分离中的跨域问题
date: 2019-07-22T08:25:32.000Z
categories:
  - vue
tags:
  - 前后端分离
  - vue
---

# vue+springboot前后端分离中的跨域问题

如今前后端分离已经成为了热门，那么本次话题就是如何解决前后端分离中的跨域问题。

首先，我们来看一下什么是前后端分离。

**前后端分离**,其实无非来说就是因为项目越来越大，从而让前端工程师和后端工程师不得不分开成为两个具体项目,二者之间只需要按照api来约定，从而大大简化了开发的工程量。

## 如何做到前后端分离

一般来说，前端可以用vue-cli搭建，它可以用webpack进行打包，并且可以利用nginx进行访问。

后端的话可以利用,spring全家桶，you know what I mean.

## 跨域问题

那么，问题来了，当我们没有用前后端分离的时候，跨域问题一般不存在，或者存在于分布式中，那时候我们可以利用js解决这个问题，但是现在我们可以在后端中着手这个问题。

解决思路:

利用springboot的filter来解决这一问题

```java
package run.app.config;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019 2019/7/8 23:43
 * Description: ://TODO ${END}
 */


import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpHeaders;
import org.springframework.web.cors.CorsUtils;
import org.springframework.web.filter.GenericFilterBean;


import javax.servlet.FilterChain;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

/**
 * Filter for CORS.
 * 解决跨域问题
 * @author johnniang
 */
@Slf4j
public class CorsFilter extends GenericFilterBean {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        log.info("进入");
        HttpServletRequest httpServletRequest = (HttpServletRequest) request;
        HttpServletResponse httpServletResponse = (HttpServletResponse) response;

        // Set customized header
        httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_ORIGIN, httpServletRequest.getHeader(HttpHeaders.ORIGIN));
        httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_HEADERS, "x-requested-with");
        httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_METHODS, "GET, POST, PUT, DELETE, OPTIONS");
        httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_CREDENTIALS, "true");
        httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_MAX_AGE, "3600");

//        todo 没看懂 好像是解决跨越问题
        if (!CorsUtils.isPreFlightRequest(httpServletRequest)) {
            chain.doFilter(httpServletRequest, httpServletResponse);
        }
    }
}
```

上面的是filter的写法，然后我们在注册中心注册一下就好了

```java


@Slf4j
@Configuration
@EnableConfigurationProperties(AppProperties.class)
public class FilterConfiguration {

    @Bean
    public FilterRegistrationBean<CorsFilter> initCorsFilter(){
        FilterRegistrationBean<CorsFilter> corsFilter = new FilterRegistrationBean<>();

        corsFilter.setOrder(Ordered.HIGHEST_PRECEDENCE);
        corsFilter.setFilter(new CorsFilter());

        corsFilter.addUrlPatterns("/*");

        return corsFilter;
    }
}

```

## 后期修改(重要)

由于vue默认请求编码格式是content-type为application/x-www-form-urlencoded的数据。 但是现在有一个需求是在post请求的时候将content-type修改为application/json; charset=utf-8格式，并且后台利用@RequestBody获取到参数，而且我们都知道这个参数并不能处理application/x-www-form-urlencoded的数据。 但是当我利用上面的过滤器的时候，还是出现了禁止访问的现象，跨域问题仍然存在，仔细查看发现是我并不允许application/json格式的对象，所以上面的filter中的请求头应该改成

```java
httpServletResponse.setHeader(HttpHeaders.ACCESS_CONTROL_ALLOW_HEADERS, "Origin, X-Requested-With, Content-Type, Accept");
```

现在问题解决。

## 现象分析

如果你打开你的控制台会发现，你的跨域请求会发出两次或者三次，但是只有一次是你真正想要的

这是你真正的请求 ![](https://s2.ax1x.com/2019/07/22/eCyCy4.png)

下面的多出来的请求

![](https://s2.ax1x.com/2019/07/22/eCy10A.png)

## OPTIONS分析

那么，这个OPTIONS是什么呢？ 经过filter发现，我们是允许这个OPTIONS请求的，后来经过google发现， W3C规范：跨域请求中，分为简单请求和复杂请求；所谓的简单请求，需要满足如下几个条件：

* GET，HEAD，POST请求中的一种；
* 除了浏览器在请求头上增加的信息（如Connection，User-Agent等），开发者仅可以增加Accept，Accept-Language，Content-Type等；
* Content-Type的取值必须是以下三个：application/x-www-form-urlencoded，multipart/form-data，text/plain。

在发出复杂请求的之前，就会出现一次OPTIONS请求。 OPTIONS请求可以被称作一次嗅探请求，通过这个方法，客户端可以在采取具体的资源请求之前，决定对资源采取何种必要措施。
