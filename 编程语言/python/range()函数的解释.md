# range()函数的解释

> range(start,stop,step)


* [start,stop) 包左不包右 如果只有一个数字，那就是是[0,stop)
* step 每一次的步长 默认为1


下面这个就相当于range(0,6,1) 
```python
    a = range(6)  # list(a): [0, 1, 2, 3, 4, 5]
```


## 逆向输出

像是下面的语句 从5-> -1 步长是-1 那就是逆输出 不包括-1,那就是从0为止
```python
#  # [5, 4, 3, 2, 1, 0]
for i in range(5, -1, -1):
    print i 
```
