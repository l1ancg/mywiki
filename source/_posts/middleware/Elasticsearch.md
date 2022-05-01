---
title: Elasticsearch 核心知识点
date: 2021-10-12 23:49:57
tags: 
  - backend
  - middleware
  - elasticsearch
---
## 特点

水平扩容、PB、可用性

协调节点：Coordinating Node（路由功能，默认都是）

专用协调节点：Dedicated Coordinating Node

## 避免脑裂

-   7.0前需要设置仲裁数量，票数大于仲裁数量时才能成功选举，仲裁参数为`discovery.zen.minimum_master_nodes`，应该大于可被选为主节点数量的一半2/3，3/5等
-   7.0后移出参数，自动仲裁

## 分片

-   主分片数量过大时单个分片数据量小，影响性能，过小时，受限于单机存储上限，需要重建索引，
-   分片离线时，会将副本设为主分片，主分片上线时成为副本
-   分片数不能修改，修改分片数量后，hash 路由算法无法路由到正确的分片上

## 故障转移

-   分片与副本在不同的主机上，当分片离线时，使用副本当做分片来达到故障转移的效果
-   分片与副本在不同的主机上，需要集群支持
-   主机上下线后，自动整理分片与副本的分布情况到 green 状态

## 健康状态

-   健康：所有分片和副本都可用
-   亚健康：所有分片都可用，部分副本不可用
-   不健康：部分分片不可用

## 文档与分片

-   对文档的id（默认）进行 hash 取模均匀的路由到不同的分片上
-   自定义路由数值`?routing=bigdata`

## 文档更新

1.  请求到`Coordinating Node`（请求处理节点），给路由到文档的分片节点上
2.  分片删除索引，重建索引，处理结果返回给`Coordinating Node`，然后返回给用户

## 文档删除

1.  请求到`Coordinating Node`，路由到主分片节点上
2.  主分片节点删除分片，调用副本节点删除副本
3.  执行结果返回给`Coordinating Node`，然后返回给用户

## 分片生命周期

### Refresh
-   存储数据时，先放入`index buffer`（内存缓冲区）
-   `refresh`操作耗时，所以同时写入`transication log`（本地文件，可以查询）
-   将`Index buffer`写入`segment`叫做`refresh，不执行`fsync`操作
-   lucene 中多个`segment`就是 es 中的`shard`
-   触发条件1：频率1秒1次，参数 index.refresh_interval
-   触发条件2：`Index buffer`被写满触发`refresh`，默认 jvm 的 10%
-   数据refresh后，就可以被搜索
-   断电时`index buffer`数据丢失，`transication log`文件得以保留，数据不会丢失

### flush

-   调用`refresh`，清空`index buffer`，清空`transication log`，
-   触发条件1：默认30分钟1次
-   `transication log`被写满，默认512m

### merge

-   segment 很多，定期合并
-   真正删除已删除的文档（lucene 中的.del文件）
-   手动 merge：POST index/_forcemerge

## 查询过程

### query（阶段1）

1.  处理请求节点在分片副本中随机选择，发送查询请求
2.  每个分片进行查询、排序，返回`from + size`个文档id，和排序值（分数）

### fetch（阶段2）

1.  对结果进行重新排序，选取从`from`到from + size`的文档id
2.  执行`multi get`获取详细文档数据

### 问题

-   `_search`查询时，可以通过`"explain": true`进行查询分析

-   性能：每个分片需要查的文档个数为`from + size`，一共需要查的文档个数为`numbers_of_shard * (from + size)`，深度分页性能挑战

-   相关性算分：各个节点的算分相互独立，打分偏离，当文档总数很少，分片越多算分越不准

    -   解决一：文档总数少时，分片数量设置为1
    -   解决二：指定参数`_search?search_type=dfs_query_then_fetch`，把每个分片的文档信息进行汇总，对所有分片的数据进行一次完整的相关性算分，损耗cpu及内存，性能低下，不建议使用

## 排序过程

-   原始内容进行，倒排无法使用，使用正排索引
-   通过id快速获取字段原始内容
-   类型一：Fielddata（手动开启、内存中（快）、动态创建、内存开销大）：mapping中的properties字段参数`fielddata: true`进行打开
-   类型二：Doc Value（默认、磁盘中、随着倒排一起创建、并且对 text 类型无效），mapping中的properties字段参数`doc_value: false`进行关闭，打开需要重建索引，明确知道不排序和聚合分析时可以关闭

## 分页

-   分页上线10000，from + size 不许超过10000，否则报错

-   search_after：深度分页，传入上一次排序查询结果的返回id，利用上一次查询结果查询下一页，无法指定页数

-   scroll：`_search?scroll?5m`，指定时间创建快照，创建后新数据无法查询，传入上一次结果的`scroll_id`

## 搜索分页场景

-   普通查询，直接查询
-   需要全部文档，例如导出，使用`scroll`
-   分页`from + size`，页数大时性能开销大
-   深度分页，使用`search_after`，无法指定页数

## 并发控制

-   PUT product/_doc/1?if_seq_no=1&if_primary_term=1

    修改`文档1`，需要满足`seq_no`字段等于0，并且`primary_term`等于1，这两个值是查出来的

-   PUT product/_doc/1?version=100&version_type=external

    修改`文档1`，需要满足`version`等于100，版本号类型属于`外部版本号`，比如数据库维护的乐观锁

## 坑
 **Cannot allocate memory**

docker 运行内存限制，无法给es足够的内存空间，调大docker内存的限制值，或者调小es的jvm内存值

**unknown setting [discovery.seed_host] please check that any required plugins are installed, or check the breaking changes documentation for removed setting**

一个机器启动多个es时，端口不同，seed_host参数需要指定9300对应的端口

## Docker运行

### 创建网卡

```
 # 所有需要互相访问的程序需要放置在同一个网络中
 docker network create \
       --driver bridge \
       --subnet 192.168.0.1/16 \
       --gateway 192.168.3.1 es-net
```
### 直接启动

```
 # --env "discovery.type=single-node"  
 # 单节点简单启动
 docker run --detach \
       --name es01 \
       --publish 9200:9200 \
       --publish 9300:9300 \
       --env "discovery.type=single-node"  
       --env "ES_JAVA_OPTS=-Xms128m -Xmx128m" \
       --ulimit nofile=65535:65535 \
       --net es-net \
       docker.elastic.co/elasticsearch/elasticsearch:7.8.1
 ​
```

### docker-compose

```
 version: '2.2'
 services:
   es01:
     image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
     container_name: es01
     environment:
       - node.name=es01
       - cluster.name=es-docker-cluster
       - discovery.seed_hosts=es02,es03
       - cluster.initial_master_nodes=es01,es02,es03
       - bootstrap.memory_lock=true
       - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
     ulimits:
       memlock:
         soft: -1
         hard: -1
     volumes:
       - data01:/usr/share/elasticsearch/data
     ports:
       - 9200:9200
     networks:
       - esnet
   es02:
     image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
     container_name: es02
     environment:
       - node.name=es02
       - cluster.name=es-docker-cluster
       - discovery.seed_hosts=es01,es03
       - cluster.initial_master_nodes=es01,es02,es03
       - bootstrap.memory_lock=true
       - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
     ulimits:
       memlock:
         soft: -1
         hard: -1
     volumes:
       - data02:/usr/share/elasticsearch/data
     networks:
       - esnet
   es03:
     image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
     container_name: es03
     environment:
       - node.name=es03
       - cluster.name=es-docker-cluster
       - discovery.seed_hosts=es01,es02
       - cluster.initial_master_nodes=es01,es02,es03
       - bootstrap.memory_lock=true
       - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
     ulimits:
       memlock:
         soft: -1
         hard: -1
     volumes:
       - data03:/usr/share/elasticsearch/data
     networks:
       - esnet    
 ​
 volumes:
   data01:
     driver: local
   data02:
     driver: local
   data03:
     driver: local
 ​
 networks:
   esnet:
     driver: bridge
```

### IK中文分词器

```
 ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.8.1/elasticsearch-analysis-ik-7.8.1.zip
```

### 设置允许跨域

在elasticserch容器中的/usr/share/elasticsearch/config/elasticsearch.yml文件中追加

```
 http.cors.enabled: true
 http.cors.allow-origin: "*"
```

### 监控软件

```
  docker run --detach --name cerebro --publish 9000:9000 --net es-net lmenezes/cerebro
```

启动好后，因为和es在同一个网络下，可以通过`http://es01:9200`进行监控
