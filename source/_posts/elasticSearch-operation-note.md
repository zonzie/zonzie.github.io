---
title: elasticSearch操作笔记
date: 2018-09-18 21:43:10
tags:
    - elasticSearch
categories: elasticSearch
description: es操作笔记
---

#### es中的一些基本概念
##### Near RealTime(NRT近实时)
elasticSearch是一个近实时的搜索平台, 从索引一个文档开始直到它可以被查询会有轻微的延迟

##### cluster(集群)
cluster是一个或者多个节点的集合, 它们一起保存数据并且提供所有的节点联合索引以及搜索功能. 集群存在一个唯一的名字身份且默认为"elasticSearch". 这个名字很重要, 如果节点安装时, 通过自己的名字加入到集群中的话, 那么一个节点只能是一个集群的一部分

##### Node(节点)
节点是一个单独的服务器, 是集群的一部分, 存储数据, 参与集群中的索引和搜索功能, 节点可以单独运行, 形成单节点集群

##### index(索引)
index是具有稍微类似特征文档的集合. 比如一个消费者数据的索引, 产品目录的索引, 订单数据的索引. 索引通过名字(小写)来标识, 并且名字在对document(文档)执行indexing(索引), search(搜索), update(更新), delete(删除)操作时会涉及到. 一个单独的集群中, 可以定义想要的索引

##### Type(类型)
在index(索引)中, 可以定义一个或者多个类型. 一个类型是索引中的一个逻辑的种类/分区,语义完全取决于自己. 一般情况下, 一个类型被定义成一组常见的文档. 假设我们在一个单独的索引中存储了一个博客平台的所有数据, 在这个索引中, 可能定义了一个用户数据类, 博客数据类型和评论数据类型

##### Document(文档)
document是索引信息的基本单位. 例如存储customer数据的文档,存储product数据的文档,存储order数据的文档

##### Shards & Replicas (分片 & 副本)
索引可以存储大量的数据, 可以超过单个节点的硬件限制. 为了解决这个问题, elasticSearch提供了把index(索引)拆分到多个Shard(分片)中的能力. 在创建索引时, 可以简单的定义Shard(分片)的数量. 每个Shard本身就是一个fully-functional(全功能的)和独立的"index", Shard可以存储到集群中的任何节点

###### Sharding(分片) 非常重要的理由:
- 水平的拆分/扩展
- 分布式的并行跨Shard操作(可能在多个节点), 从而提高了性能/吞吐量
每个索引可以被拆分成多个分片, 一个索引可以0个或者多个副本,开启副本后, 将会有主分片和副本分片,分片和副本的数量可以在索引被创建时指定,也可以在任何时候修改副本的数量, 但是不能修改分片的数量
默认情况下, es中的每个索引分配了5个主分片和1个副本,如果集群中有两个节点, 则会有5个主分片和5个副本分片

#### 基本的一些命令
##### 查看集群的健康程度
`curl -XGET 'localhost:9200/_cat/health?v&pretty'`

##### 获取集群节点的列表
`curl -XGET 'localhost:9200/_cat/nodes?v&pretty'`

##### 列出所有的索引
`curl -XGET 'localhost:9200/_cat/indices?v&pretty'`

##### 创建索引customer
`curl -XPUT 'localhost:9200/customer?pretty&pretty'`

##### 创建一个customer文档到customer索引中, 'external'类型, ID为1
```bash
curl -H "Content-Type:application/json" -XPUT 'localhost:9200/customer/external/1?pretty&pretty' -d `
{
    "name": "john doe"
}`
```

##### 删除索引
`curl -XDELETE 'localhost:9200/customer?pretty&pretty'`

##### 不指定id可以用POST请求创建一个新的文档,id会自动生成
```bash
curl -H "Content-Type:application/json" -XPOST 'localhost:9200/customer/external?pretty&pretty' -d'
{
    "name": "John Doe"
}`
```

##### 更新文档
使用POST方法,修改name,添加age字段
```bash
curl -H "Content-Type:application/json" -XPOST 'localhost:9200/customer/external/1/_update?pretty&pretty' -d '

    "doc": {
        "name": "Jane Doe",
        "age": 20
    }
}'
```

使用PUT方法,修改name,添加age字段
```bash
curl -H "Content-Type:application/json" -XPUT 'localhost:9200/customer/external/1/_update?pretty&pretty' -d '
{
    "name": "james Doe"
}'
```

使用脚本更新年龄
```bash
curl -H "Content-Type:application/json" -XPOST 'localhost:9200/customer/external/1/_udpate?pretty&pretty' -d '
{
    "script": "ctx._source.age += 5"
}'
```

##### 删除文档
```
curl -XDELETE 'localhost:9200/customer/external/2?pretty&pretty'
```

##### 批量操作
批量插入数据
```bash
curl -H "Content-Type:application/json" -XPOST '192.168.198.88:9200/customer/external/_bulk?pretty&pretty' -d '
{"index": {"_id": "5"}}
{"name": "john dow"}
{"index": {"_id": "6"}}
{"name": "jane wick"}
'
```

批量更新数据
```bash
curl -H "Content-Type:application/json" -XPUT '192.168.198.88:9200/customer/external/_bulk?pretty&pretty' -d '
{"index": {"_id": "5"}}
{"name": "john dow"}
{"index": {"_id": "6"}}
{"name": "jane wick"}
'
```

更新id=5的文档, 删除id=6的文档
```bash
curl -H "Content-Type:application/json" -XPOST '192.168.198.88:9200/customer/external/_bulk?pretty&pretty' -d '
{"update": {"_id":"5"}}
{"doc": {"name": "John doe becomes Jane Doe"}}
{"delete": {"_id":"6"}}
'
```

#### 数据探索部分
使用REST reqeust URI发送搜索参数<br/>
查询index为customer下的所有的文档
```bash
curl -XGET 'localhost:9200/customer/_search?pretty'
```

使用REST request body发送请求, 使用POST或者GET, 一般浏览器会默认抛弃掉GET请求的请求体, 使用POST比较保险
```bash
curl -H "Content-Type:application/json" -XPOST 'localhost:9200/customer/_search?pretty' -d '
{
    "query": {"match_all": {}}
}'
```
##### 返回的数据主要有以下几部分:
- took: es执行搜索的耗时(ms)
- time_out: 搜索是否超时
- _shards: 告诉我们搜索了多少分片,统计了成功/失败的分片
- hits: 搜索的结果
- hits.hits: 实际的搜索结果数组(默认为前10的文档)
- sort: 结果的排序key(没有则按照score排序)
- score

##### DSL(Domain-Specific Language, 领域特定语言)
elasticSearch提供了可执行查询的Json风格的DSL. 这个查询语言非常全面,我们从基础开始<br/>
查询customer中的一条数据
```bash
curl -H "Content-Type:application/json" -XGET '192.168.198.88:9200/customer/_search?pretty' -d '
{
    "query": {
        "match_all": {}
    }
}'
```
添加查询的分页,from:指定文档开始的编号,size:返回的条数,sort:指定排序的字段和规则
```bash
curl -H "Content-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '
{
    "query": {"match_all": {}},
    "from": 10,
    "size": 10,
    "sort": {
        "age": {
            "order": "desc"
        }
    }
}'
```

返回指定字段name和age
```bash
curl -H "Content-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '
{
    "query": {
        "match_all": {}
    }
    "_source": ["name", "age"]
}'
```

返回年龄为20的customer
```bash
curl -H "Content-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '
{
    "query": {
        "match": {
            "age": 20
        }
    }
}'
```

返回所有name中包含john的customer, 不是精确匹配
```bash
curl -H "Content-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '{
    "query": {
        "match": {
            "name": "john"
        }
    }
}'
```

精确匹配一个单词或者短语
```bash
curl -H "Content-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '{
    "query": {"match_phrase": {
        "name": "john"
    }}
}'
```

###### bool查询
使用boolean逻辑构建较小的查询到更大的查询中去<br/>
返回name中同时包含john和jane的文档
```bash
curl -H "Contetn-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '
    "query": {
        "bool": {
            "must": [
                {"match": {"name": "john"}},
                {"match": {"name": "jane"}}
            ]
        }
    }
}'
```
上面的例子中, bool must 语句指定了所有的查询必须为true时匹配到的文档<br/>
如果两个名字是或的关系,例子如下
```bash
curl -H "Contetn-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '{
    "query": {
        "bool": {
            "should": [
                {"match": {"name": "john"}},
                {"match": {"name": "jane"}}
            ]
        }
    }
}'
```
bool should中只要有一个为true就会匹配到<br>
如果两者都不想匹配到
```bash
curl -H "Content-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '{
    "query": {
        "bool": {
            "must_not": [
                {"match": {"name": "john"}},
                {"match": {"name": "jane"}}
            ]
        }
    }
}'
```

以上如果条件都不为true才会匹配到文档<br/>
我们可以在bool查询中同时联合使用must, should, must_not
```bash
curl -H "Content-Type:application/json" -XGET 'localhost:9200/customer/_search?pretty' -d '{
    "query": {
        "bool": {
            "must": [
                {"match": {"age": 40}}
            ],
            "must_not": [
                {"match": {"name": "jane"}}
            ]
        }
    }
}'
```

###### 过滤查询

