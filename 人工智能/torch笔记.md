# torch笔记

## DataSet

[自定义DataSet](https://zhuanlan.zhihu.com/p/105507334)

## 超参的训练
![img](https://img2023.cnblogs.com/blog/2286223/202305/2286223-20230530160755124-672694820.png)
上面代码想要训练超参数0.1和0.9，使用Parameter的api包裹一下，代码他是可以训练的Ternsor

## moudleList和seqList的区别
[知乎博客](https://zhuanlan.zhihu.com/p/75206669)

## 对dim和axis的理解
[参考博客](https://blog.csdn.net/znsoft/article/details/104384409) 


## Adam套路代码


```python

torch.optim.Adam(model.parameters(),
                lr=0.001,
                betas=(0.9, 0.999),
                eps=1e-08,
                weight_decay=0,
                amsgrad=False)


for epoch in range(1000):
    optimizer.zero_grad() # 新的一轮将导数设置为0
     
    pred = net(x) # 过神经网络
    loss = loss_func(pred,y)
    loss.backward() # 反向传播求梯度
    optimizer.step() # 优化

```

## model.train()和model.eval()之间的区别

[参考博客](https://zhuanlan.zhihu.com/p/494060986)

## Linear线性层

定义linear(x,y).这里x是输入维度，y是输出维度。我让一个Tensor变量进入线性层，要保证自己的最里面的维度和x一样即可，其他都是批处理操作。这里的思想和多维矩阵相乘的思想一样。
## 多维矩阵相乘

**matmul** 在多维变量中，例如256,10,100,32和256,10,32,100.只有最后两个维度是矩阵相乘，也就是输出是100*100.前面的维度直接批处理。


## repeat()函数
[参考函数](https://www.cnblogs.com/vvzhang/p/16100045.html#:~:text=Pytorch%E7%9A%84repeat%E5%87%BD%E6%95%B0%20repeat%E5%8F%AF%E4%BB%A5%E5%AE%8C%E6%88%90%E6%8C%87%E5%AE%9A%E7%BB%B4%E5%BA%A6%E4%B8%8A%E7%9A%84%E5%A4%8D%E5%88%B6%20import%20torch%20a%20%3D%20torch.randn%20%283%2C,1.1322%2C%20-0.7117%5D%5D%29%20b%20%3D%20a.repeat%20%281%2C2%29%20b%2Cb.size%20%28%29)

在例子 y = x.repeat(2, 1) 中，参数 (2, 1) 表示在第一个维度上重复 2 次，在第二个维度上重复 1 次。因此，结果张量 y 在第一个维度上重复了两次，而第二个维度上没有重复。这是因为第一个维度的索引是 0，而不是 1。

```python

x = torch.tensor([1,2,3])
 
#将一维度的x扩展到三维
xx = x.repeat(4,2,1)
 
/**
扩展步骤如下(倒着执行)：
1  最后一个维度1：此时将[1,2,3]中的数字直接重复1次，得到[1,2,3]，保持没变
2  倒数第二个维度2：先将上一步骤的结果增加一个维度，得到[[1,2,3]]，然后将最外层中括号中的整体重复2次，得到[[1,2,3],[1,2,3]]
3  倒数第三个维度4：先将上一步骤的结果增加一个维度，得到[[[1,2,3],[1,2,3]]]，然后将最外层中括号中的整体重复4次，得到[[[1,2,3],[1,2,3]],[[1,2,3],[1,2,3]],[[1,2,3],[1,2,3]],[[1,2,3],[1,2,3]]]
4  三个维度扩展结束，得到结果。
 
**/
```

## mask_fill函数
一般用作掩码用，这里[参考博客](https://blog.csdn.net/jianyingyao7658/article/details/103382654)


## 转置之后需要转换Tensor的shape

转置完之后需要使用contiguous()方法使Tensor在内存中是连续的然后才可以使用view()函数。
不然就会报错
