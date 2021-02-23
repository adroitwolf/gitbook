---
title: springboot 2.x 采用监控模块
date: 2019-11-05 19:51:35
categories:
  - springboot
tags:
  - springboot
---

# springboot + actuator + prometheus + grafana 实现资源监控


项目逐渐成型起来了，但是如果打包到服务器上之后，自己不可能时时刻刻关注服务器的状态，这时候舍友小A在一旁和我吐槽：
你不用想的很复杂，直接在网上找一个可以时时刻刻监控你自己服务器上的cpu啥的，一有事直接给你发邮件不就好了。

一语点醒梦中人，说干就干

百度了一下，好像对于spring boot 2.x最火的版本就是actuator + prometheus + grafana。但是网上的东西，你懂的，云龙混杂。所以，属于自己的才是最好的，接下来就和你们分享一下如何利用上面的数据搭建这个东西，由于博主现在还没有学docker，所以所有的东西都将在window上运行，之后学了docker在补上(咕咕咕)


## springboot

首先对我们的springboot下手，先贴一下pom包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
            <version>1.1.1</version>
  </dependency>
```


然后贴一下application.yaml配置文件

```yaml
# 系统监控
management:
  endpoint:
    metrics:
      enabled: true
    health:
      show-details: always
  metrics:
    export:
      prometheus:
        enabled: true

  endpoints:
    web:
      base-path: /monitor
      exposure:
        include: prometheus
```


> 注意里面的 endpoints.web.base-path  这个可以自定义
默认是/actuator



启动我们的应用

访问 http://HOST:PORT/monitor

我们可以看到这样的数据
```json

{"_links":{"self":{"href":"http://localhost:8099/monitor","templated":false},"prometheus":{"href":"http://localhost:8099/monitor/prometheus","templated":false}}}
```
我们打开 http://HOST:PORT/monitor/prometheus 网址 可以看到一大坨数据，好了，配置成功了



## prometheus

贴一下网址 https://prometheus.io/

下载去上面的官网，官网里面有文档来说明这个软件，本篇博客不多叙述

安装成功后，修改里面的**prometheus.yml**文件

```yml
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'eblog'
    scrape_interval: 15s
    metrics_path: /monitor/prometheus
    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:8099']
```

我们需要修改的就是job_name(项目名) + metrics_path/targets(网址)


然后在terminal上输入 prometheus.exe --config.file=prometheus.yml

访问 http://localhost:9090

如果出现下面的界面，就表示我们配置成功了
![](https://s2.ax1x.com/2019/11/05/M9Fkyn.png)

## grafana
别说别的,上官网 https://grafana.com/

下载完毕后，请观察[官网文档](https://grafana.com/docs/installation/windows/)的这些话，配置你的grafana
![](https://s2.ax1x.com/2019/11/05/M9F2tS.png)

启动你的软件，然后访问 http://localhost:3000/
登陆成功后创建你的data source 和 dashboard
![](https://s2.ax1x.com/2019/11/05/M9F5Xn.png)

由于我们使用的是prometheus，所以我们选择这一项
![](https://s2.ax1x.com/2019/11/05/M9FxXR.png)


接着我们配置他的configure

![](https://s2.ax1x.com/2019/11/05/M9kA9e.png)

最后save&Test，保存成功！
之后我们需要添加图表，按照下图操作

![](https://s2.ax1x.com/2019/11/05/M9kDCF.png)

![](https://s2.ax1x.com/2019/11/05/M9kogH.png)

![](https://s2.ax1x.com/2019/11/05/M9kqbt.png)

![](https://s2.ax1x.com/2019/11/05/M9APrn.png)
出现数据，配置完成！
