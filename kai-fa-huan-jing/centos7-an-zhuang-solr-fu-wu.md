---
title: centos7安装solr服务
date: 2019-04-12T10:54:33.000Z
categories:
  - linux
tags:
  - 安装教程
  - 搜索服务
  - linux
  - java
---

# centos7 下安装solr服务

## 1. 安装java环境

***

* 将jdk扔到linux下，解压
* 修改环境变量

```
# 打开配置文件
vim /etc/profile

# 在文件末尾添加
JAVA_HOME=/usr/local/java/jdk1.8.0_201
CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar
PATH=$PATH:$JAVA_HOME/bin
export JAVA_HOME CLASSPATH PATH

#保存并关闭

source /etc/profile
```

## 2. 安装solr

***

1. 下载solr安装包 \[官方网址]:(http://www.apache.org/dyn/closer.lua/lucene/solr/8.0.0)
2. 扔到linux下，解压
3. cd到目录下\bin

```
 ./solr start -force #默认启动8983端口
```

> 注意 ，这里我并没有修改文件限制，linux会限制传输文件大小 ![](https://s2.ax1x.com/2019/04/12/AbujvF.png)

1. 关闭防火墙

```
systemctl stop firewalld
```

1. 打开游览器，访问 192.168.xx.xx.8983

## 3.solr功能介绍

***

![](https://s2.ax1x.com/2019/04/12/AbajBt.png)

![](https://s2.ax1x.com/2019/04/12/Abdp4S.png)

## 4. 配置solr

***

### 配置core环境

***

* 打开solr-admin
* 按照下图操作 ![](https://s2.ax1x.com/2019/04/12/AbKrxU.png)
* 之后会提示错误，因为找不到config和schema文件
* 打开linux，在%安装目录%/server/configsets/\_default/下的conf文件copy到 %安装目录%/server/%core名称%/ 下
* 重新创建core
* 看到自己的core文件夹下面出现这几个文件，证明安装成功 ![](https://s2.ax1x.com/2019/04/12/AbMCLQ.png)

### 配置中文分词器

***

> 其实solr8.0已经自带了中文分词器 ![](https://s2.ax1x.com/2019/04/12/AbMuyF.png)

*   但是为了业务要求，我们使用IK分词器

    ik可以去maven仓库去下载

    ![](https://s2.ax1x.com/2019/04/12/AbtyfH.png)
* 将三个jar包放到%solr安装包%/server/solr-webapp/webapp/WEB-INF/lib/下
* 将剩余的三个文件放到%solr安装包%/server/solr-webapp/webapp/WEB-INF/classes/下\[如果没有，自己新建]
* 配置ik 打开%自己CORE%/conf/managed-schema.xml文件

```

    <!-- ik分词器 -->
        <fieldType name="text_ik" class="solr.TextField">
        <!-- 索引分词器 -->
        <analyzer type="index" isMaxWordLength="false" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
        <!-- 查询分词器 -->
        <analyzer type="query" isMaxWordLength="true" class="org.wltea.analyzer.lucene.IKAnalyzer"/>
        </fieldType>
```

* 重启 ./solr restart -force

### 配置业务域FileType

***

打开%自己CORE%/conf/managed-schema.xml文件

```
<field name="item_title" type="text_en" indexed="true" stored="true"/>
<field name="item_sell_point" type="text_en" indexed="true" stored="true"/>
<field name="item_price"  type="plong" indexed="true" stored="true"/>
<field name="item_image" type="string" indexed="false" stored="true" />
<field name="item_category_name" type="string" indexed="true" stored="true" />
<field name="item_desc" type="text_en" indexed="true" stored="false" />

<field name="item_keywords" type="text_en" indexed="true" stored="false" multiValued="true"/>
<copyField source="item_title" dest="item_keywords"/>
<copyField source="item_sell_point" dest="item_keywords"/>
```

重启 ./solr restart -forces
