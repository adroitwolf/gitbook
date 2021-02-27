---
title: java 算法心得
date: 2021-01-09 10:52:34
tags:
- java
- 算法

categories:
- 数据结构
---

# java 算法心得

## 选择排序与冒泡排序之间的关系

这几天学习算法，发现了一个有意思的现象。选择排序与冒泡排序的时间复杂度都是O(n^2)

并且呢，冒泡排序是两两相邻的去比较。而选择排序是一次比较所有。


```java

/**
    * 功能描述: 选择排序算法
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



## 为什么HashMap的时间复杂度可以为O(1)



