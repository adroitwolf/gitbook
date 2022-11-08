# tensorflow笔记



## kernel_initializer
[参考博客](https://www.cnblogs.com/Renyi-Fan/p/13703476.html)
示例代码：
```python
Dense(64,
        kernel_initializer='random_uniform',
        bias_initializer='zeros')

```

这个意思就是用什么方法去初始化weight

## bias_initializer

和上面一样就是用什么方法初始化bias