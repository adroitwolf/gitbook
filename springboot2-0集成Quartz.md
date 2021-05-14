---
title: springboot2.0集成Quartz
date: 2019-11-29 14:20:45
categories:
- springboot
tags:
- Quartz
- springboot
---

# Spring Boot 2.0 集成Quartz

这几天制作博客的文章点击量的任务,我们需要定时将redis的缓存存到数据库，所以定时任务调度是绝对少不了的了。

## Quartz

先说一下Quartz的核心架构吧
![](https://s2.ax1x.com/2019/11/29/QkbOII.png)

我们要清楚里面的重点

Job-> 我们需要让Quartz做些什么
JobDetail -> Quartz完成什么任务，任务名称等详细信息
Trigger-> Quartz多久调度一次什么任务



## 集成

导入jar包

```xml
<!--定时任务-->
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-quartz</artifactId>
      </dependency>
```

写定时任务
> 定时任务的关键就是需要继承QuartzJobBean类，然后实现executeInternal方法

```java

import lombok.extern.slf4j.Slf4j;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.quartz.QuartzJobBean;
import run.app.entity.model.BlogStatus;
import run.app.service.BlogStatusService;

import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019 2019/11/29 14:06
 * Description: 文章点击量定时任务
 */
@Slf4j
public class ClickBlogTask extends QuartzJobBean {

    @Autowired
    BlogStatusService blogStatusService;

    private SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-ss hh:mm:ss");

    @Override
    protected void executeInternal(JobExecutionContext jobExecutionContext) throws JobExecutionException {
        log.info("ClickTask收录-------- {}", sdf.format(new Date()));
        blogStatusService.transClikedCountFromRedis2DB();
    }
}

```

写Quartz配置文件

```java


import org.quartz.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import run.app.listner.ClickBlogTask;

/**
 * Created with IntelliJ IDEA.
 * User: WHOAMI
 * Time: 2019 2019/11/29 14:03
 * Description: Quartz配置文件
 */
@Configuration
public class QuartzConfig {
    private static final String CLICK_TASK_IDENTITY = "ClickTaskQuartz";

    @Bean
    public JobDetail clickQuartzDetail(){
        return JobBuilder.newJob(ClickBlogTask.class).withIdentity(CLICK_TASK_IDENTITY).build();
    }


    @Bean
    public Trigger clickQuartzTrigger(){
        SimpleScheduleBuilder scheduleBuilder = SimpleScheduleBuilder.simpleSchedule()
                .withIntervalInSeconds(10) //10c/s
                .repeatForever();  //一直执行
        return TriggerBuilder.newTrigger().forJob(clickQuartzDetail())
                .withIdentity(CLICK_TASK_IDENTITY)
                .withSchedule(scheduleBuilder)
                .build();
    }

}

```
