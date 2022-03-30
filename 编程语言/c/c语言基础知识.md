# c语言基础知识

## 循环输入直到输入空

ACM的基础题：
```C
    int a,b;
    while(~scanf("%d %d",&a,&b)){

    };
```
这边是直到用户输入 ctrl+z(意为文件结束操作符)敲回车之后 结束循环



##  浮点数输出操作
%m.nf m表示 至少要输出多少字符(其中小数点也代表一个字符)，不够的需要在前面补充空格。n表示需要保留多少小数。

```c
    float f = 2.11;
    printf("%5.1f",f);


    //输出  3.2,前面有两个空格
```

## 四舍五入

1. 利用基础语句
```c
    float f  = 3.2;
    int i = (int)(f+0.5);
```

2. 利用math头文件里面的round函数
```c
    #include <math.h>

    float f = 3.4;
    int i = (int)round(f);
```



## ASCII码


char字符都是通过判断ASCII码来认知符号的，所以我们可以通过对ASCII码进行操作来对字符进行基础的加密，俗称凯撒密码。

> 字符A~Z的ASCII码为65~90，字符a~z的ASCII码为97~122，每个大小写字符的ASCII码都相差32


下面是凯撒密码代码
```c
char s= 'c',encode = (s+4-'a')%26+'a';
printf("encode:%c\n",encode);
printf("decode:%c",(s -4 - 'a' + 26)%26 + 'a');
```


### malloc 分配问题

https://www.cnblogs.com/npyx/p/3564968.html