---
title: java里面的Set集合坑
date: 2019-11-29T23:24:58.000Z
categories:
  - java
tags:
  - java
  - Set
---

# java里面的Set集合坑

Set的定义已经很清楚了，他的集合里面是不允许有重复变量的。 所以,Set更适合集合去重。

但是在最近我做每日最热文章的时候，出现了一个小bug，就是在redis里面取数据的时候，被继承的父类是不能被写入数据的。

下面看我的例子

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public final class PopularBlog extend BlogStatus{
    String blogName;


}

@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public class BlogStatus {
    private Long id;

    private Integer clickcount;

}
```

redis这样是取不出来id和clickcount的，但是我的blogName故意设置的有一样的数据，所以就会直接被去重。

所以，必须放弃继承，改写成这样

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@ToString
public final class PopularBlog {
    String blogName;

    private Long id;

    private Integer clickcount;
}
```
