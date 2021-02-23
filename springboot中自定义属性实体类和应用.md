---
title: springboot中自定义属性实体类和应用
date: 2019-06-30 10:37:52
categories:
- springboot
tags:
- springboot
---

# springboot中自定义属性实体类和应用

我们在当初学习SSM的时候学习过@Value这个属性，可以将配置文件中的属性加载到想要加载的类中，这个当然是一个可行的办法,但是如果属性有很多一直写@Value也是一件很麻烦的事情，现在我们可以用**@ConfigurationProperties**和**@EnableConfigurationProperties**这两个注解完成这个问题。


## 具体应用

* 配置文件如下:

```yaml
whoami:
  auto-scan: false
  auto-del: true

```


上面的配置文件就相当于一个例子，没有什么作用
下面我们写自定义的属性类

* 属性类：

@ConfigurationProperties(prefix = "whoami")的作用就是将whoami的前缀的配置加载到这个实体类中

```java
@Data
@ConfigurationProperties("whoami")
public class AppProperties {
    private Boolean autoDel = false;

    private Boolean autoScan = true;

}

```


> 在这里我们可以看出来，实体类里面的属性默认值和配置文件正相反，而且配置文件的属性写法是'-'风格，而自定义文件里面的是驼峰写法，其实只要能对应就完全没问题。



* 验证一下是否导入

先说一下@EnableConfigurationProperties的作用，其实就是让@ConfigurationProperties生效


```java
@Configuration
@EnableConfigurationProperties(AppProperties.class)
public class FilterConfiguration {

    @Autowired
    AppProperties appProperties;

    @Bean
    public FilterRegistrationBean<LogFilter> initLogFilter(){
        log.info("属性配置；{}",appProperties.getAutoDel());
        ...

```

上面的代码是一个配置类，我们可以看到，我们在初始化一个过滤器的时候顺便打印了AutoDel这个属性，，运行一下，可以看到打印台的信息


![](https://s2.ax1x.com/2019/06/30/ZlXeRs.png)

在这里我们可以看到，打印出来的是我们配置文件的信息。
