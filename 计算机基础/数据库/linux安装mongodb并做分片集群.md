# linux安装mongodb并做分片集群

## 安装mongodb

我们先去[官网](https://www.mongodb.com/try/download/community)下载对应的版本，我这里选择的是4.0.0 tgz版本

> 在linux下解压打开

```bash

sudo tar -zxvf mongodb-linux-x86_64-4.0.0.tgz -C /usr/local/  # 解压至指定文件目录

sudo mv /usr/local/mongodb-linux-x86_64-4.0.0 /usr/local/mongodb # 改名字

```

> 编写环境变量

打开自己prfile文件(我这里是zsh，所以我这里直接对.zshrc修改)
```bash
vim .zshrc

# 在后面添加变量
export MONGODB_HOME=/usr/local/mongodb
export PATH=$PATH:$MONGODB_HOME/bin

source .zshrc # 这里看你的profile文件是什么
```

## mongodb设置

我这里设置了三个分片服务器(没有集群)
两个config服务器(官方要求必须是集群模式)
一个router服务器

具体信息如下

| 名字        | port  | 集群     |
| :---        |    :----:   |          ---: |
| shard0      | 27100       | Nan   |
| shard1   | 27101        | Nan      |
| shard2   | 27102        | Nan      |
| config1   | 27200        | config  |
| config2  | 27201        | config  |
| r2  | 8848    | Nan  |

### 新建db和log文件夹

```bash
# 创建响应的db
mkdir -p ~/db/s0 
mkdir -p ~/db/s1
mkdir -p ~/db/s2
mkdir -p ~/db/c0
mkdir -p ~/db/c1

# 创建log 文件夹
mkdir ~/log

```

> 创建log日志
```bash
touch ~/log/s0.log
touch ~/log/s1.log
touch ~/log/s2.log
touch ~/log/c0.log
touch ~/log/c1.log
touch ~/log/r.log

```


### 编写conf文件

> shard config文件编写

```conf
# 这边三个分片服务器仅仅是 这三个配置看着改
port= 27100 
logpath= /home/kali/log/s0.log
dbpath=/home/kali/db/s0/

logappend=true
maxConns=100
storageEngine=wiredTiger
bind_ip = 0.0.0.0

shardsvr=true # 这个是标记它为分片服务器

replSet=shard1 # 这个标记认定他是一个集群

```

**这里我们的分片服务器虽然只有一个，但是必须得是集群模式，不然事务模式无法开启**

> config 文件编写

```conf
port= 27200
logpath= /home/kali/log/c1.log
dbpath=/home/kali/db/c1/
logappend=true
# fork=true
maxConns=100
# auth=true
storageEngine=wiredTiger
bind_ip = 0.0.0.0
replSet = config # 标记它为复制集
# keyFile = /home/kali/mongodb/keyfile

configsvr = true # 这个是标记它为配置服务器
```

> 配置路由服务器

```conf
# shard 1
port= 8848
logpath= /home/kali/log/r.log
logappend=true

configdb =config/192.168.17.17:27200,192.168.17.17:27201
maxConns=100
# auth=true
bind_ip = 0.0.0.0

```

### mongodb 启动

> 启动三个分片服务器

```bash
mongod -f ./mongdb_s{0/1/2}.conf --fork

use admin # 准备配置服务器集群设置

rs.initiate({_id:"shard1",members:[{_id:0,host:"192.168.17.17:27100"}]})
``` 

> 启动两个配置服务器

```bash
mongod -f ./mongdb_c{0/1}.conf --fork # 启动服务

mongo --port 27200 # 进入某一个线程内部

use admin # 准备配置服务器集群设置

rs.initiate({_id:"config",members:[{_id:0,host:"192.168.17.17:27200"},{_id:1,host:"192.168.17.17:27201"}]})
# 看到mongo>界面编程子节点表示初始化成功

rs.status() # 查看复制群是否成功
```

> 启动路由服务器


```bash

mongos -f ./mongdb_r.conf --fork

#进入该线程内部

mongo --port 8848

use admin

sh.addShard("shard1/192.168.17.17:27100")
sh.addShard("shard2/192.168.17.17:27101")
sh.addShard("shard3/192.168.17.17:27102")
```

## 实现分片功能


> 还在路由服务器内部


> 设置chunk
```bash
use config
db.setting.save({"_id":"chunksize","value":1}) # 设置块大小为1M是方便实验，不然需要插入海量数据
```

* 这里我们设置test数据库里面的user表来分片
```bash
use test

sh.enableSharding("test")

db.user.createIndex({"id":1}) # 以"id"作为索引

sh.shardCollection("test.user",{"id":"hashed"}) # hash分片

```

> 往数据库中插数据 测试50k条


```bash

for(i=1;i<=50000;i++){db.user.insert({"id":i,"name":"jack"+i})} #模拟往calon数据库的user表写入5万数据
```


## 集群启动顺序

1. 启动config server
2. 启动router server
3. 启动shard server