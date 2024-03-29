---
title: 算法心得
date: '2021-01-09T10:52:34.000Z'
tags:
  - java
  - c++
  - 算法
categories:
  - 数据结构
---

# 算法心得

## 选择排序与冒泡排序之间的关系

这几天学习算法，发现了一个有意思的现象。选择排序与冒泡排序的时间复杂度都是O\(n^2\)

并且呢，冒泡排序是两两相邻的去比较。而选择排序是一次比较所有。

```java
/**
    * 功能描述: 选择排序算法  --> int[] 是引用传递，即使不return input的指也会改变
    * @Param: [input] 
    * @Return: int[]
    * @Author: ADROITWOLF
    * @Date: 2021/2/27 21:33
     */
    public static int[] selectSort(int [] input){
        int min = 0;
        for (int i =0;i<input.length;i++ ){
            min = i;
            for(int j = i+1;j<input.length;j++){ 
                min = input[min] > input[j] ? j:min;
            }
            if(i != min){ //说明最小值已经是那啥了
                int tmp = input[i];
                input[i] = input[min];
                input[min] = tmp;

            }
        }
        return input;
    }



    /**
    * 功能描述: 冒泡排序
    * @Param: [input]
    * @Return: int[]
    * @Author: ADROITWOLF
    * @Date: 2021/2/27 21:56
     */
    public static int[] bubbleSort(int [] input){
        for (int i=0;i<input.length;i++){
            for(int j=0;j<input.length-1;j++){
                if(input[j] > input[j+1]){
                    int tmp = input[j];
                    input [j] = input[j+1];
                    input[j+1] = tmp;
                }
            }
        }
        return input;
    }
```

## 为什么HashMap的时间复杂度可以为O\(1\)

通过《算法图解》这本书得知，散列表的查找时间复杂度是O\(1\)。具体实现方法是将输入数据和索引坐标做一个映射，并且散列表是通过数组来实现存储的，数组的查找时间复杂度为O\(1\)。HashMap的底层实现原理也是利用的散列表，所以他的时间复杂度为O\(1\).

并且，HashMap的使用极为广泛： 1. 用作去重 2. 作为缓存 3. 映射查找遍历。

## 尾递归

尾递归说白了就是，在函数的后面直接return 自己，且没有任何的计算操作。举个例子：

```java
public int recursion(int a){
    if xxx{
        return 1;
    }else{
        return recursion(a*x);
    }
}
```

> 我们都知道递归会将函数压栈，不仅消耗大量的时间，而且有很大的不确定性，稳定性无法保证。 尾递归，则是编译器自动的将不重复调用栈，而是直接覆盖函数栈，使空间复杂度从O\(n\)变成了O\(1\)

