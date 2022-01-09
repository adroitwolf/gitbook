---
title: scrapy初探
date: 2019-11-05T12:52:15.000Z
categories:
  - 爬虫
tags:
  - 爬虫
---

# scrapy初探

## scrapy 框架下爬取网站图片思想

最近博客项目也做的差不多了，但是总感觉和别人的比差了点什么，突然有一天想起来了。对啊！我们没有图源啊。 但是，按照我的脾气来说，别人的不如自己的，与其将链接指向他们的网站，不如自己画\~很显然不现实。 最后，我打算拾起大一放弃的python爬虫，我干脆直接在别人的网站上面爬一些就好喽。 大一的时候，之所以放弃爬虫单纯以为python做不了什么大项目，现在看来是我单纯了，人家python可以直接装载到docker里面啊！唉，流下了无知的泪水。 好了，废话说到这里，我们开始爬虫教程。

## 准备环境

首先要确定自己有相应的模块，也就是scrapy。其实里面还需要另一个模块来着，我给忘了，小事情，只要少什么下什么就好了。 具体命令

```bash
pip install scrapy
```

创建工程同样也很简单，这里我们一笔带过

```bash
# 切换到你想要保存的路径下
scrapy startproject app(项目名称)
```

### 框架简单的入门

![](https://s2.ax1x.com/2019/11/05/MpG64H.png)

创建工程之后，自动生成了这样的文件，画红线的不要在意。

下面分析一下工程目录结构： spider/\*\* -> 里面的就是我们的爬虫文件,一开始只有init.py文件其他的都是自己创建的，反正改动最多的就是这里 init.py-> 无视，不需要在意

items.py-> 很重要，我们爬取的东西需要有的POJO存取啊，但是我们不能乱创建POJO文件，只能在items里面创建，这是规定

middlewares.py -> 新手教程关，暂时请无视

piplines.py -> 可以无视，里面是写一些存放策略，就是我把一些东西都保存到实体里面了，我想要下载啊，或者把这些存放到数据库里面啊，piplines就是干的这种任务，但是,我为什么说可以无视呢，因为我这里只讲下载图片，图片的话scrapy有默认的pipline(翻译成管道) setting.py -> 可以设置一些爬虫的规则，比如说，请求的header需要加的参数啊，或者是否支持重连啊，还有需要启用的pipline啊(这里认为最重要！)

> 这上面的内容很重要，请仔细查看

### 分析爬取的网站

我这里拿一个美女图片网站分析吧 ![](https://s2.ax1x.com/2019/11/05/MpUPw6.png) 我们可以看到，这里的网站首页有图集，然后图集有a标签指向具体图片，而且点击每个图集后，可以看到每个图集的网址都是有规则的 ![](https://s2.ax1x.com/2019/11/05/MpUDtU.png) 就是example.com/archives/xxx.html

里面的xxx是随机的数字

然后里面都是我们想要爬取的小姐姐

总结一下：

网址： 静态网址 反爬措施： 不知道，有也不会讲怎么应对，下一篇讲 目标集中网址：example.com/archives/xxx.html

这样就简单了

### 创建第一只爬虫

创建爬虫命令

有两种方式，一种是创建一个基于crawl模板的，一种是不基于模板的，我们先开始第一支

```bash
scrapy genspider -t crawl grilSpider(爬虫名字) example.com (爬取的网址)
```

\-t很显然是template的意思

总之，我们创建了一个爬虫，下面我就开始给他创建规则，就是最起码告诉他怎么爬

talk is cheap,show your code:

```python
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule
from ..items import AppItem


class GirlspiderSpider(CrawlSpider):
    name = 'girlSpider'
    allowed_domains = ['example.com']
    start_urls = ['http://example.com']

    rules = (
        Rule(LinkExtractor(allow=r'http://example.com/archives/\d+.html'), callback='parse_item', follow=False),
    )

    def parse_item(self, response):
        print(response.url)
        item = AppItem()
        item['image_urls'] =response.xpath(".//div[@id='post-content']/p/img/@src").extract()
        yield item
```

下面讲解这些东西是什么意思，首先看上面的import模块，和你们的唯一的不同就是多了最后一行，那个你现在可以忽视，我们一会说

name 就是我们的爬虫名称 allowed\_domains 就是爬虫允许爬取的域名，总不能让他爬着爬到了别人的网站里面了吧

start\_urls 里面存放着我们需要爬取的起始网址，每次都会做一个递归爬取。

rules -> 这个可算是crawl 里面的重点了 Rule(LinkExtractor(allow=r'http://example.com/archives/\d+.html'), callback='parse\_item', follow=False)

首先我们的网址放在allow属性里，但是你可以看到里面的网址是我们想要爬取的 根据上面分析的来说我们需要爬取的就是这种网页，后面调用parse\_item函数，这个follow是是否开启递归的意思，就是如果里面如果有下一页的话，我们需要继续从这一页开启跳转到下一页之后接着爬这里由于那个网站很简单，所以就设置成False

> 上面我们可以总结成根据http://example.com 开始，爬取http://example.com/archives/\d+.html 这种网址来分析和爬取

#### items.py

这里我们开始创建我们的图片实体，由于我们下载器用scrapy默认的 scrapy.pipelines.images.ImagesPipeline下载器，所以我们的实体就是默认的两个属性 image\_urls -> 爬取图片的网址，images -> 有就行，不用管

下面是我们这个爬虫需要用到的实体类

```python
class AppItem(scrapy.Item):
    image_urls = scrapy.Field()
    images = scrapy.Field()
```

然后我们将这个方法引入到我们的那个爬虫文件里面中，这也就是最后的那句import的作用

同时我们需要在setting.py打开我们的下载器

#### setting.py

```python
ITEM_PIPELINES = {
   #将下载器放在这里面后面的数字是数序，1000以内的，你随便写
    'scrapy.pipelines.images.ImagesPipeline':300,
}

# 这个就是那个默认下载器必须要有的，就是图片存储位置，img是文件夹名称，你随便定一个就好
IMAGES_STORE = 'img'
```

#### 爬虫.py

好了，完成了一系列任务，现在开始解析网页了，这里我直接用xpath语法，不懂的可以取百度了

```python
def parse_item(self, response):
    print(response.url)
    item = AppItem()
    item['image_urls'] =response.xpath(".//div[@id='post-content']/p/img/@src").extract()
    yield item
```

首先我们实例化一下上面创建的item的函数

同时我们需要穿进去我们需要爬取的image的url就好了，直接网页定位到图片然后获取所有符合条件的网址，直接给image\_urls就好了，下面的yield你把它看成return就可以了

### 运行爬虫

```bash
scrapy crawl girlSpider
```

执行完毕之后，我们看到我们的爬取的小姐姐了

![](https://s2.ax1x.com/2019/11/05/Mpw73F.png)
