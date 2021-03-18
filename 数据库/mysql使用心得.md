---
title: mysql使用心得
date: 2019-05-13 21:52:45
categories:
- mysql
tags:
- mysql
---

# mysql使用心得

* 对于那种datetime属性的默认值 可以写CURRENT_TIMESTAMP，来填充当前时间。

## 对于Join连接的经验

首先，连接的结果可以在逻辑上看作是由SELECT语句指定的列组成的新表。

左连接与右连接的左右指的是以两张表中的哪一张为基准，它们都是外连接。

外连接就好像是为非基准表添加了一行全为空值的万能行，用来与基准表中找不到匹配的行进行匹配。假设两个没有空值的表进行左连接，左表是基准表，左表的所有行都出现在结果中，右表则可能因为无法与基准表匹配而出现是空值的字段。


下面的图：
[![rO59w6.png](https://s3.ax1x.com/2020/12/30/rO59w6.png)](https://imgchr.com/i/rO59w6)

> 在使用JOIN语句的时候，肯定会用到 ON关键字，这个字段的意思是 将两个表合成一张表的条件，而有的时候又会使用到where关键字，这个关键字则是在组合成临时表之后的筛选。

```sql
SELECT column_name(s)
FROM table1
LEFT OUTER JOIN table2
ON table1.column_name=table2.column_name;
```

这里面 的ON关键字就是用来变成一张临时表的条件.

## UNION语句

首先说一下区别，JOIN使用起来是将结果扩充表的列，相当于纵向；而UNION使用起来必须保证SELECT出来的表结构都是一样的，最后将两个表的结果纵向连接在一起。

* UNION 中有去重操作是UNION ALL 和SELECT DISNCT差不多。

* UNION 中如果使用ORDER BY 字段 必须在**最后一张**表中使用。

