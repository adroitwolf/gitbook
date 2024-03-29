# numpy 常用语句

## 初始化数据
[np.ones](https://blog.csdn.net/silent1cat/article/details/119883549)

## numpy用来升维
[np.newaxis的作用](https://blog.csdn.net/THMAIL/article/details/121762644)

通常当我们使用最简单的x-y线性模型去训练神经网络的时候,当你输入x的值的时候，tensorflow会直接报错，因为你必须输入(1,)类型的数据，这样升维就变得很关键,下面的代码可以让300个列表变成[300,1]格式的数据


```python
    x_data = np.linspace(-1,1,300)[:,np.newaxis]
```

## reshape()函数
[reshape](https://zhuanlan.zhihu.com/p/385818806)


## 归一化
* np.linspace(-1,1,300)

在-1到1之间平均分出300个数


## 标准化
```python
np.random.normal
```

[正态分布函数](https://blog.csdn.net/wzy628810/article/details/103807829)

## shape
之前一直对shape函数有隔阂，这里做一个总结


* shape = () 代表里面就是一个数
* shape = (2,)  => array([2,3])一个一维数组


## 随机选取

> np.random.sample()

> np.random.choice(target,p)

## ...号

比如说，我创建一个a = np.arrange(60).reshape(3,4,5)的数组，我想选最后一维度的第2个元素，我可以使用 a[:,:,2]。
但是当维度高的时候，可以使用a[...,2]