---
title: java自定义注解
date: 2019-06-04T23:07:05.000Z
categories:
  - java
tags:
  - springboot
---

# java自定义注解

***

这次的文章是我在上次文章的AOP文章遇到一个坑，通过joinPoint得到签名后，获得到他的注解为空，后来发现自己没有加相应的注解，这里记录此次的坑。

首先我们看一下正确的注释代码

```java

@Target(ElementType.METHOD)
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
public @interface Cache {

    boolean ifDelete() default false;
}

```

上次我们的文章的中本来是没有@Target等注解的，但是后来发现，这几个注解的作用是非常大的。

## 每个注解的作用

**@Documented**

这个注解的作用就是，在生成javadoc文档的时候，将用过这个这个注解的方法的文档里面也写上这个注解。

**@Retention(Retention.RUNTIME)**

这个注解的作用在我看来就是在编译java代码的时候将这个注解也加上，也就是为什么不加这个注解，我们利用getAnnotation()这个方法等到的注解为null了。

**Target()**

这个注解是我们这个注解是为谁服务的，比如我们上面写的ElementType.METHOD,就是决定我们是为了方法服务的，如果写在类上面，这个注解会失效。

ElementType后面还有其他类别:

* ANNOTATION\_TYPE 注解类型声明
* CONSTRUCTOR 构造方法声明
* FIELD 字段声明（包括枚举常量）
* LOCAL\_VARIABLE 局部变量声明
* METHOD 方法声明
* PACKAGE 包声明
* PARAMETER 参数声明
* TYPE 类、接口（包括注解类型）或枚举声明
* TYPE\_PARAMETER @since 1.8
* TYPE\_USE @since 1.8

**@Inherited** 这个注解表示，打上这个注解之后，这个类如果被其他类继承后，子类会自动打上这个注解。
