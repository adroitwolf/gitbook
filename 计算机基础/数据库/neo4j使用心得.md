# neo4j使用心得
neo4j我们需要关注的主要是两个部分即实体集和关系集。
## 导入文件
我们可以使用neo4j-admin工具将我们经过python等编程语言数据清洗后的结构文件直接导入，一般导入的文件的格式是csv。
一般的格式如下：
> 实体csv格式


| person_id:ID(Executive)          | name   | gender | age:int | :LABEL |
| -------------------------------- | ------ | ------ | ------- | ------ |
| 1 | 杜玉岱 | 男     | 58      | Person |
| 2 | 延万华 | 男     | 45      | Person |

* person_id 是我们自定义的ID的名字，并且在:ID()中有值，是方便我们在导入关系的时候方便指定ID
* name，gender：这两列都是我们自定义的实体的属性，可以有多个
* age:int 这一列代表我们要求输入的类型是int类型的，默认为string类型
* :LABEL 该列代表着实体的类型


> 关系csv格式

| :START_ID(Executive)             | jobs          | :END_ID(Stock) | :TYPE     |
| -------------------------------- | ------------- | -------------- | --------- |
| 1 | 董事长/董事   | 1         | employ_of |
| 2 | 副董事长/董事 | 2         | employ_of |
| 2 | 副董事长/董事 | 1         | employ_of |



* :START_ID(Executive)  我们需要在Executive空间里面找头节点
* jobs： 该列是我们自定义的属性，可以有多个
* :END_ID(Stock) 我们需要在Stock找尾节点
* :TYPE 代表着两个实体之间的关系类型


在上面的例子中，假设我们已经有了Executive.csv和stock.py两个实体集文件和relation.csv关系集文件，我们需要使用neo4j-admin将数据导入到数据库，具体命令为：

```bash
neo4j-admin import --id-type=STRING --nodes executive.csv --nodes stock.csv --relationships relation.csv --database neo4j
```

* --database是指定将数据导入哪里的数据库




## 创建实体

如果我们创建一个上文中csv中的实体，命令如下所示。
> 注意，在Cypher中，()代表一个实体， () -[]->() 代表 从那个节点连接到另一个节点

```bash
CREATE (:Person {person_id:"1",name: "杜玉岱", gender:"男",age: 58})
CREATE (:Company {stock_id:"1",name: "金枫酒业", code:"600616"})

```
* 我们发现age属性我们没有使用引号，所以数据库将会识别age属性为int型


## 创建关系

> 我们首先需要查找到需要连接的两个的实体，并分别使用自定义名称标识它们，即下文总的node1和node2
```bash
match (node1:Person {person_id:"1"}),(node2:Company {stock_id:"1"})
create (node1) -[:employ_of {jobs:"董事长/董事"}] -> (node2)

```


## 查询语句

### 查询实体
```bash
match (node:Person) where node.name = "杜玉岱" return node.age,node.gender
```
我们使用match语句查询，并且根据属性确定节点，当然我们可以返回节点的所有属性，或者直接返回节点，如果是返回节点的话直接return node即可


### 查询关系
下面这条语句是查询所有Person连接Company中属于employ_of的关系，当然我们可以通过添加where等关键词对其进行过滤操作，当然，也可以根据修改return后面的语句来返回关系的一些属性操作
```bash
match (:Person) -[r:employ_of]->(:Company) return r
```
结果如下图所示
![img](https://img2023.cnblogs.com/blog/2286223/202308/2286223-20230812224858871-94293908.png)

### 查询二跳邻居

这里为了方便，我根据上文多创建了一个关系，如下图所示：
![img](https://img2023.cnblogs.com/blog/2286223/202308/2286223-20230812224744200-2115471336.png)
我们想要找到杜玉岱的二跳邻居，也就是延万华，查询语句为：
```bash

match (:Person {name:"杜玉岱"}) -[]-> (:Company) -[]-> (end:Person) return end

```


当然，我们可以不断的通过()-[]->()-[]->()查询三跳或者更多的节点