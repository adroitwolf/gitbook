---
title: mysql 利用触发器来实现check约束
date: 2019-05-17 18:11:51
categories:
- mysql
tags:
- mysql
---

# mysql 利用触发器来实现check约束

------

由于我用的是mysql5.6.34版本，所以据我的了解来看，mysql对于check约束还是一个摆设。

但是我们可以利用触发器(trigger)来实现一种类check约束

下面我们来一个实例：

------

我们需要创建一个学生表 sno 主键，sage 年龄在16-20之内

这个sage就是一个check约束

首先我们创建我们的studen表

```
CREATE TABLE Student (
	Sno VARCHAR(3) PRIMARY KEY,
	Sage INT ,
)ENGINE=INNODB DEFAULT CHARSET=utf8;
```



接着我们创建触发器

```
# 这个是在插入insert之前触发 指定是在student表
CREATE TRIGGER test_student_insert_check BEFORE INSERT
ON Student FOR EACH ROW
BEGIN
	DECLARE msg varchar(100);
	IF NEW.Sage <= 16 OR NEW.Sage >= 20 
	THEN
		SET msg = CONCAT('无效的年龄!');
		SIGNAL SQLSTATE 'HY000' SET MESSAGE_TEXT = msg;
	END IF;
END;
```

> 注意，虽然这样可以做到check的效果，但是还是非常不建议用触发器，因为太慢了，所以我们最好还是在插入之前判断，例如spring的service层。

> 而且，这个插入只是管理insert插入的，更新的话还是不会拦截的，而且我们又不可能在创建一个update触发器，太浪费资源，所以这个博文只是一种思想，并不实用。