# javaSE基础知识点汇总

***

## java数据类型

八大基础数据类型： byte,char,short,int,long,float,double,boolean

> 包装类型是类/对象，默认是Null,而int这种的基础数据类型默认为0(牛客中出现过)

## String 详解

### 内存问题

```java

A:{
    String a = "a";
    String b = "a";
}

B:{
    String a = new String("a");
    String b = new String("a");
}
```

上面两个代码块，有区别没？ 答案肯定是有的，A代码块的执行原理是这样的

首先在自己线程的栈帧中生成一个引用型变量a,然后再到堆里面的常量池中寻找有没有a的对象，如果有就把地址直接转向那里，如果没有，新建；b也是同理。

> 注意，java中只要是那种包装数据类型都是创建一个相当于指针的东西。

**提示**这个是包装数据类型，如果是char这种的呢？则是直接存在栈中

B代码块你就考虑只要是new，那肯定是直接再heap里面直接创建一个新的对象啦。

## java编译

java编译命令: javac xxx.java

编译后会生成class文件，生成的文件个数和源程序中类的**个数一致**。

## java值传递和引用传递

网上的教程大多都是参差不齐，这里整理一下我自己的思路。

值传递： 基本数据类型

引用传递： String等各种对象，数组。

## Java中Stream的使用

### forEach

这个我们肯定特别熟悉，遍历嘛。但是这里有一个注意的点，Collection包下面的集合都已经实现了forEach方法，也就是说，我们不需要将List对象先转换成stream对象，再用forEach方法

### map

map对象，他可以做一个对数据的处理，最后return出来的那个东西变成了一个steam流,举个例子

```java

List<User> users;
 // 这里是个有好多数据的user列表，我们想吧里面的属性转出两个来怎么办？直接用map

// 假设UserChange里面就一个id和name,其他的都删掉了
 List<UserChange> lists = users.stream().map(item->{
     xxxxx //去掉那些属性
     return A;
 }).collect(Collections.toList());
```

### filter

这个是筛选出来符合filter里面条件的list，这里不举例了

## final关键字的理解

final如果赋给了变量，证明这个变量是不能修改的 但是，要注意，如果给数组附上了final关键字，他的数组元素是可以修改的

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

\== 是直接比较地址，一般用来比较8大基本数据类型

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

## JAVA加载顺序

父类静态代码块>> 子类静态代码块>> 父类代码块>> 父类构造函数>> 子类代码块>> 子类构造函数

* 可以利用反射机制Class.forName("包名")理解class的消息

## JVM的垃圾回收机制

finalize finalize是Object的方法，所有类都继承了该方法。 当一个对象满足垃圾回收的条件，并且被回收的时候，其finalize()方法就会被调用

两种常见的回收机制：

1. 定时回收 每隔30分钟进行一次回收，这种机制的弊端是如果垃圾产生的比较快，有可能30分钟之内垃圾已经把内存占用光了，导致性能变慢
2. 当垃圾占到某个百分比的时候，进行回收 比如，当垃圾占到70%的时候，进行回收。 这种机制的弊端是，如果垃圾产生的频率很快，那么JVM就必须高频率的进行垃圾回收。 而在垃圾回收的过程中， JVM会停顿下来，只做垃圾回收，而影响业务功能的正常运行。

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

## String,StringBuilder,StringBuffer之间的区别

String要注意他是一个final，也就是说不可继承的一个常量。

String Builder线程不安全，但是效率高

String buffer 线程安全，效率低

## Vector,ArrayList之间的区别

* 共同点： vector和arraylist创建一个固定大小的空间
* 不同点：

Vector线程安全,arrayList不安全；每次容量满了之后,arrayList容量扩张50%,Vector扩张一倍。

## 继承

子类一旦override父类方法之后，即使向上转型成为父类，调用的仍然是子类的方法。