# mybtatis添加数据返回主键

今天心血来潮，想用自己的项目来发布一些博客，可是当我提交成功后，想要访问的时候，突然告诉我找不到？！？
吓得我赶紧去查看一下数据库日志，结果发现主键对不上。
是这样，我的博客是放在数据库中的，而为了方便，我又将博客分成了两个表，一个表存放博客的基本信息，一个存放博客的内容信息，插入的时候，先存放基本信息返回主键id，然后将id和内容插入到内容表中，本来是没有问题的，后来才发现，逆向工程的insert的语句，返回的不是主键id，而是影响的行数。问题找到了，那么解决他就很简单了。

## 解决办法

经过查阅资料发现，主键id是可以返回的，但是会返回给你当初insert进去的对象里面的属性中，不会返回。这也是当然的吧，万一主键约束有多个不就瞎了，你咋返回？

但是默认的insert是不会返回的，必须保证你的数据库主键是设置成了逐步递增的，而且，还需要下面三个属性：

* useGeneratedKeys="true" (必要)
* keyProperty="xx" (必要，你想要返回给对象的属性名称)
* keycolumn="xx" (非必要，这个是指定数据库中那个是主键)


将这三个属性写在xml的方法上，像是这样
```xml
  <insert id="insertSelective" parameterType="run.app.entity.model.Blog" useGeneratedKeys="true" keyProperty="id" >
```

然后你的主键就会返回给你这个Blog对象了，大功告成。


## tkmybatis解决办法

因为tkmybatis易于上手的特性，后面我的项目大多都是用tkmybatis来开发

```java
@Table(name = "user")
@Data
@ToString
@Builder
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String name;

}

```
只要在我自己的Bean上面添加@GeneratedValue的注释后，在我执行insert语句会自动将ID填充到id变量中