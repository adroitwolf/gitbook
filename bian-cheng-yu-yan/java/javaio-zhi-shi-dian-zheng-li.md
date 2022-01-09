---
title: java.io知识点整理
date: 2019-05-23 22:01:13
categories:
- java
tags:
- java
---

# java io 流的一些知识点整理
------

## java流类图总结
![](https://s2.ax1x.com/2019/05/23/VPsDmt.png)

<!-- more -->

## 字节流和字符流的区别

字节流(1byte 8bit)处理的是二进制文件，也就是说,二进制文件什么也能处理，比如文字和图片视频什么的。
而字符流(1char,2byte,16bit)则只能处理文本类型，但是它可以处理编码问题，例如我们的字节流读取出来的文件是乱码的，就是因为文件的编码问题。

>下面是一个读取中文文本的样例。

```java
public static void main(String args[]){
        int b; //这里的默认值是0
        try {
          //这里的f变量是为了后面建立char[]数组读取文件大小
            File f = new File("D:\\\\test.txt");
            //创建字符流
            InputStreamReader isr = new InputStreamReader(new FileInputStream(f),"GBK");
            //建立buf ,注意，如果文件中有中文的话，这里的buf里面会多建立空间，因为中文占两个字节空间
            char[] buf = new char[(int) f.length()];
            int len = isr.read(buf);
            String rs = new String(buf,0,len);
            isr.close();
        }
        catch (Exception e) {
            System.out.println("出错了，原因是：");
            System.out.println (e.toString());
        }

    }
```
