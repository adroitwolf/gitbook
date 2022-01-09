---
title: iview中table的使用经验
date: 2019-07-26 21:22:41
categories:
- iview
tags:
- 前端
---

# iview中table的使用经验


由于iview官网上面的讲解真的是非常的简单，所以像是我这种阿库娅用户真的很不友好，所以这个博客是讲解iview的Table如何和springboot进行后端交互，类似bootStrap-table的那种吧，注意这里没有用render去渲染，因为我是真的不会啊！！！


## 前端代码框架编写


```html
<Table :columns="managerColumns" :data="articleData" stripe >
            <div slot-scope="{ row, index }" slot="action">
                <Button type="primary"  style="margin-right: 5px">编辑</Button>
                <Button type="error"  >删除</Button>
            </div>
        </Table>
        <!-- 分页列表 -->
        <div style="margin: 10px;overflow: hidden">

            <div style="float: right;">
                <Page :total="total"  :current="pageNum" @on-change="changepage"
                @on-page-size-change="_nowPageSize"
                show-total show-sizer></Page>
            </div>
```


在前端渲染的是这种样子 =>


![](https://s2.ax1x.com/2019/07/26/eu11gO.png)


这里简明的说明里面的属性吧

首先是Table标签里面的columns是 表头属性  后台的数据是这样的
```json
managerColumns: [{
      title: '标题',
      key: 'title',

  },
  {
      title: '更新时间',
      key: 'nearestModifyDate',
  },
  {
      title: "操作",
      slot: 'action'
  }

],
```


然后是data属性，这个就是我们需要从后台需要渲染的数据了。

在<Table>标签里面，我们如果需要进行操作的话，需要用到solt-scope这个属性，具体我也不太清楚，只能说是：可以对应表头的一个东西，其他的照抄就完事了。
需要注意的是，如果你用到了slot，那个表头的json一定不能用key,而是用slot.


接下来就是分页的按钮们了


样式不需要动，唯一确定的是page里面的属性

total => 其实就是你从后台获取到的数据总量

current => 这个是现在你所在的页数
