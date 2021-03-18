---
title: 记录springboot利用pagehepler失效的原因
date: 2019-07-28 21:01:15
categroies:
- springboot
tags:

- pagehepler
- Mybatis
- springboot
---

# 记录springboot利用pagehepler失效的原因

首先说一下记录这个博客的原因吧，之前用SSM的时候，也用过pageHepler，但是一切也OK,后来打算利用springboot搭建项目的时候在数据量小的时候(也就是数据还没有到分页的时候)也没又出现问题，但是当数据量慢慢增多的时候突然发现了一件事，就是每次利用pageHepler发过去的数据，数据其实传的是全部！！！！

吓得我赶紧后台debug了一下，没有任何异常原因，经过查询作者的github项目主页的时候发现那么几个错误解决办法，希望对此有帮助。


## pagehepler失效，查询全部数据

这里我的Springboot是2.1.4版本，Mybatis和pageHelper版本在下面
```xml
<dependency>
           <groupId>com.github.pagehelper</groupId>
           <artifactId>pagehelper-spring-boot-starter</artifactId>
           <version>1.2.12</version>
       </dependency>

       <dependency>
           <groupId>org.mybatis.spring.boot</groupId>
           <artifactId>mybatis-spring-boot-starter</artifactId>
           <version>2.0.1</version>
       </dependency>
```

在一开始，我查询的博客大多都是说什么版本不对啊，或者没有配置属性啊，这里我澄清一点，版本问题确实有，但是充其量是pagehepler版本太低的问题。而且，版本冲突的时候springboot本身就会报错！还有就是属性问题，根据我的测试，(本人mysql数据库)直接上默认属性就行，根本不需要任何配置。所以说那些博客真的很浪费大家时间。


那么，这个失效的原因在哪里呢。经过反复的排查，发现了作者这句话，就是PageHepler.startstartPage(pageNum,pageSize);这句话，必须要后面紧跟查询语句，不能插任何逻辑语句，这就很蛋疼了，因为我只前利用SSM的时候就直接在函数的开头写上这句话，防止啥时候忘了。
然后加上这句话的时候，返回的数据正常了。。

正当我以为这个问题解决了的时候，随之出现了第二个bug....

## pageHepler返回的total不正确

当我兴奋的打开我的网站乱搞的时候，突然发现，数据倒是对了，但是返回值完全对不上，根本就是当前页面的数据总量。吓得我赶紧去翻sql日志，结果日志也显示我已经查询过了。

得，还得去作者主页。。。

经过盘了一下主页发现，原来是我查询完了之后，不想暴露一些关键属性，比如外键啊啥的，就利用lambda得stream进行了一次流操作。具体逻辑在这


```java

// 查询前期操作
        Integer blogger_id = userService.getUserIdByToken(token);

        BlogExample blogExample = new BlogExample();
        BlogExample.Criteria criteria = blogExample.createCriteria();
        criteria.andBloggerIdEqualTo(blogger_id);


        // 开始查询
        PageHelper.startPage(pageNum,pageSize);
        List<BlogWithBLOBs> blogWithBLOBs = blogMapper.selectByExampleWithBLOBs(blogExample);

// 查询到结果并且处理
        blogWithBLOBs =  blogWithBLOBs.stream().map(item->{return new BlogWithBLOBs(item.getId()
                ,item.getStatus(),
                item.getTitle(),
                item.getSummary(),
                item.getReleaseDate(),
                item.getNearestModifyDate(),
                item.getTagTitle(),
                item.getContent(),
                item.getContentMd());}).collect(Collectors.toList());

// page结构化准备分页数据
        PageInfo<BlogWithBLOBs> list = new PageInfo<>(blogWithBLOBs);

        DataGrid dataGrid = new DataGrid();

        dataGrid.setTotal(list.getTotal());

        dataGrid.setRows(list.getList());

```



但是作者的主页说过这样一句话
![](https://s2.ax1x.com/2019/07/28/e1eK4P.png)

lambda会让数据丢失，所以total就返回了默认的当前**所有数据**，这也就是为什么我们会看到total返回不正确了，这是人家默认的啊！！！

还有一点值得注意的就是，作者说我们返回的查询结果其实也是Page属性，也就是PageHepler.start的那个返回值。

那么现在解决办法就很明显了，我们可以将get的List用一个对象接收，然后对那个对象进行处理。

或者，放弃利用lambda(不可能)
