# matplotlib相关内容

## 中文标题乱码

加上下面的代码就可以，同时SimHei是字体的意思，可以替换成自己电脑中存在的字体
```python 
plt.rcParams["font.sans-serif"]=["SimHei"]
```