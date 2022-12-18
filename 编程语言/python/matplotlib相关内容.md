# matplotlib相关内容

## 中文标题乱码

加上下面的代码就可以，同时SimHei是字体的意思，可以替换成自己电脑中存在的字体
```python 
plt.rcParams["font.sans-serif"]=["SimHei"]
```

## 设置子图
有的时候我们不想要一张图的展示，我们可以使用子图的概念,其中subplot的前两个数字是构建子图的规格，第三个数字是选取第几块子图

```python
plt.figure(1)
ax1 = plt.subplot(221)
ax1.set_title()
ax1.bar()
....
```