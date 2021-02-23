---
title: javaSE基础知识点
date: 2019-05-24 10:51:47
categories:
- java
tags:
- javaSE
---

# javaSE基础知识点汇总
------

## final关键字的理解

final如果赋给了变量，证明这个变量是不能修改的
但是，要注意，如果给数组附上了final关键字，他的数组元素是可以修改的

例如：
```java
public static void main(String args[]){
        final  int b[] = {5};
        int a[] = {3};
        b[0] = 2;
        // b = a;  编译错误，因为是final类型
        System.out.println(b[0]);
    }
```

## equals 和 == 之间的区别

== 是直接比较地址，一般用来比较8大基本数据类型

equals则是一般由开发人员复写的，但是如果默认，则会比较他们的值是否一致，equals的源代码是：

```java
public boolean equals(Object obj) {
    return (this == obj);
}

```

> 但是需要注意一点的是 java中对String的equals写法是这样的,他复写了原始的方法



```java

@Override
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = count;
        if (n == anotherString.count) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = offset;
            int j = anotherString.offset;
            while (n-- != 0) {
                if (v1[i++] != v2[j++])
                    return false;
            }
            return true;
        }
    }
    return false;
}
```


## Java 标记跳出外循环

```java
public static void main(String[] args) {
        int i,j;
        A:for(i =0 ; i<10;i++){
            B:for(j = 0 ;j<10;j++){
                break A;
            }
        }
    }

```


标记如上文所示，在外层循环上面写上A标记，在内循环里面直接Break 标记，起到goto语句的作用。


### JAVA加载顺序

父类静态代码块>> 子类静态代码块>> 父类代码块>> 父类构造函数>> 子类代码块>> 子类构造函数

* 可以利用反射机制Class.forName("包名")理解class的消息



## JVM的垃圾回收机制

finalize
finalize是Object的方法，所有类都继承了该方法。 当一个对象满足垃圾回收的条件，并且被回收的时候，其finalize()方法就会被调用


两种常见的回收机制：
1. 定时回收
每隔30分钟进行一次回收，这种机制的弊端是如果垃圾产生的比较快，有可能30分钟之内垃圾已经把内存占用光了，导致性能变慢

2. 当垃圾占到某个百分比的时候，进行回收
比如，当垃圾占到70%的时候，进行回收。 这种机制的弊端是，如果垃圾产生的频率很快，那么JVM就必须高频率的进行垃圾回收。 而在垃圾回收的过程中， JVM会停顿下来，只做垃圾回收，而影响业务功能的正常运行。

一般说来 JVM会采用两种机制结合的方式进行垃圾回收。


* java特殊的内存泄漏

在下面的例子中，for创建的o 对象放入了 al里面，但是 很明显我们在之后就用不到o们了，如果al没有释放的话，这些对象会一直存在
```java

public class MemoryLeak {
    static ArrayList<Object> al = new ArrayList<Object>();
 
    public static void main(String[] args) {
 
        for (int i = 0; i < 100; i++) {
            Object o = new Object();
            al.add(o);
        }
 
    }
}
```