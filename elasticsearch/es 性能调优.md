# es 性能调优

[es调优实践]: https://cloud.tencent.com/developer/article/1156231?from=groupmessage&amp;isappinstalled=0



# 集群负载处理

## 段合并？？

每个段是一个lucene实例，合并段可以减少内存开销，lucene 索引文件一般不超过2g，不然效率很低

```
POST myindex/_forcemerge?max_num_segments=4
```

# 科学使用与问题避免

## bulk 批量写入

- 提高写入效率。

利用如下接口。

bulk请求的doc数量设定区间推荐为1k~1w。

```js
POST _bulk
{ "index" : { "_index" : "test", "_type" : "type1" } }
{ "field1" : "value1" }
{ "index" : { "_index" : "test", "_type" : "type1" } }
{ "field1" : "value2" }
```

## 调整translog同步策略

- 提交写入速度。

flush 时间，默认5s，即5s从文件系统，刷新translog到磁盘上。

调整刷新到磁盘时间，一般默认5s就行，否则，可能translog太大，刷到磁盘时间较长。

默认持久化策略是同步的，如果接受一定概率的数据丢失，可以调整为异步的。

```js
PUT my_index
{
  "settings": {
    "index": {
      "translog": {
        "sync_interval": "5s",
        "durability": "async"
      }
    }
  }
}
```

## 【重要】调整refresh_interval时间

- 提高写入速度

- 减少段数量，降低内存开销

写入lucene的数据，并不是实时可搜索的，ES必须通过refresh的过程把内存中的数据转换成Lucene的完整segment后，才可以被搜索。默认情况下，ES每一秒会refresh一次，产生一个新的segment，这样会导致产生的segment较多，从而segment merge较为频繁，系统开销较大。如果对数据的实时可见性要求较低，可以通过下面的命令提高refresh的时间间隔，降低系统开销。

```js
PUT my_index
{
  "settings": {
    "index": {
        "refresh_interval" : "30s"
    }
  }
}
```

## 写入数据不指定_id，让ES自动产生

- 提高写入效率

当用户显示指定_id写入数据时，ES会先发起查询来确定index中是否已经有相同_id的doc存在，若有则先删除原有doc再写入新doc。这样每次写入时，ES都会耗费一定的资源做查询。如果用户写入数据时不指定doc，ES则通过内部算法产生一个随机的_id，并且保证_id的唯一性，这样就可以跳过前面查询_id的步骤，提高写入效率。

所以，在不需要通过_id字段去重、update的使用场景中，写入不指定_id可以提升写入速率。腾讯云CES技术团队的测试结果显示，无_id的数据写入性能可能比有_id的高出近一倍，实际损耗和具体测试场景相关。

```js
# 写入时指定_id
POST _bulk
{ "index" : { "_index" : "test", "_type" : "type1", "_id" : "1" } }
{ "field1" : "value1" }

# 写入时不指定_id
POST _bulk
{ "index" : { "_index" : "test", "_type" : "type1" } }
{ "field1" : "value1" }
```

## 使用routing

- 提高写入效率
- 提高查询效率

对于数据量较大的index，一般会配置多个shard来分摊压力。这种场景下，一个查询会同时搜索所有的shard，然后再将各个shard的结果合并后，返回给用户。对于高并发的小查询场景，每个分片通常仅抓取极少量数据，此时查询过程中的调度开销远大于实际读取数据的开销，且查询速度取决于最慢的一个分片。开启routing功能后，ES会将routing相同的数据写入到同一个分片中（也可以是多个，由index.routing_partition_size参数控制）。如果查询时指定routing，那么ES只会查询routing指向的那个分片，可显著降低调度开销，提升查询效率。

routing的使用方式如下：

```js
# 写入
PUT my_index/my_type/1?routing=user1
{
  "title": "This is a document"
}

# 查询
GET my_index/_search?routing=user1,user2 
{
  "query": {
    "match": {
      "title": "document"
    }
  }
}
```

## 为string类型的字段选取合适的存储方式

- 提高写入效率
- 降低内存开销
- 降低存储开销

- 存为text类型的字段（string字段默认类型为text）： 做分词后存储倒排索引，支持全文检索，可以通过下面几个参数优化其存储方式： 		norms：用于在搜索时计算该doc的_score（代表这条数据与搜索条件的相关度），如果不需要评分，可以将其关闭。 		index_options：控制倒排索引中包括哪些信息（docs、freqs、positions、offsets）。对于不太注重_score/highlighting的使用场景，可以设为 docs来降低内存/磁盘资源消耗。 		fields: 用于添加子字段。对于有sort和聚合查询需求的场景，可以添加一个keyword子字段以支持这两种功能。

```js
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "title": { 
          "type": "text",
          "norms": false,
          "index_options": "docs",
          "fields": {
            "raw": { 
              "type":  "keyword"
            }
          }
        }
      }
    }
  }
}
```

- 存为keyword类型的字段： 不做分词，不支持全文检索。text分词消耗CPU资源，冗余存储keyword子字段占用存储空间。如果没有全文索引需求，只是要通过整个字段做搜索，可以设置该字段的类型为keyword，提升写入速率，降低存储成本。 设置字段类型的方法有两种：一是创建一个具体的index时，指定字段的类型；二是通过创建template，控制某一类index的字段类型。

```js
# 1. 通过mapping指定 tags 字段为keyword类型
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "tags": {
          "type":  "keyword"
        }
      }
    }
  }
}

# 2. 通过template，指定my_index*类的index，其所有string字段默认为keyword类型
PUT _template/my_template
{
    "order": 0,
    "template": "my_index*",
    "mappings": {
      "_default_": {
        "dynamic_templates": [
          {
            "strings": {
              "match_mapping_type": "string",
              "mapping": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        ]
      }
    },
    "aliases": {}
  }
```

## 查询使用query-bool-filter组合取代普通query

- 提高查询效率
- 降低集群负载

默认情况下，ES通过一定的算法计算返回的每条数据与查询语句的相关度，并通过_score字段来表征。但对于非全文索引的使用场景，用户并不care查询结果与查询条件的相关度，只是想精确的查找目标数据。此时，可以通过query-bool-filter组合来让ES不计算_score，并且尽可能的缓存filter的结果集，供后续包含相同filter的查询使用，提高查询效率。

```js
# 普通查询
POST my_index/_search
{
  "query": {
    "term" : { "user" : "Kimchy" } 
  }
}

# query-bool-filter 加速查询
POST my_index/_search
{
  "query": {
    "bool": {
      "filter": {
        "term": { "user": "Kimchy" }
      }
    }
  }
}
```

## 【重要】按需控制index的分片数和副本数

- 降低内存开销
- 提高工作效率

分片（shard）：一个ES的index由多个shard组成，每个shard承载index的一部分数据。

副本（replica）：index也可以设定副本数（number_of_replicas），也就是同一个shard有多少个备份。对于查询压力较大的index，可以考虑提高副本数（number_of_replicas），通过多个副本均摊查询压力

shard数量（number_of_shards）设置过多或过低都会引发一些问题：shard数量过多，则批量写入/查询请求被分割为过多的子写入/查询，导致该index的写入、查询拒绝率上升；对于数据量较大的inex，当其shard数量过小时，无法充分利用节点资源，造成机器资源利用率不高 或 不均衡，影响写入/查询的效率。

对于每个index的shard数量，可以根据数据总量、写入压力、节点数量等综合考量后设定，然后根据数据增长状态定期检测下shard数量是否合理。腾讯云CES技术团队的推荐方案是：

- 对于数据量较小（100GB以下）的index，往往写入压力查询压力相对较低，一般设置3~5个shard，number_of_replicas设置为1即可（也就是一主一从，共两副本） 。
- 对于数据量较大（100GB以上）的index： 		一般把单个shard的数据量控制在（20GB~50GB） 		让index压力分摊至多个节点：可通过index.routing.allocation.total_shards_per_node参数，强制限定一个节点上该index的shard数量，让shard尽量分配到不同节点上 		综合考虑整个index的shard数量，如果shard数量（不包括副本）超过50个，就很可能引发拒绝率上升的问题，此时可考虑把该index拆分为多个独立的index，分摊数据量，同时配合routing使用，降低每个查询需要访问的shard数量。

# 稳定性调优

## linux参数调优

```js
# 修改系统资源限制
# 单用户可以打开的最大文件数量，可以设置为官方推荐的65536或更大些
echo "* - nofile 655360" >>/etc/security/limits.conf
# 单用户内存地址空间
echo "* - as unlimited" >>/etc/security/limits.conf
# 单用户线程数
echo "* - nproc 2056474" >>/etc/security/limits.conf
# 单用户文件大小
echo "* - fsize unlimited" >>/etc/security/limits.conf
# 单用户锁定内存
echo "* - memlock unlimited" >>/etc/security/limits.conf

# 单进程可以使用的最大map内存区域数量
echo "vm.max_map_count = 655300" >>/etc/sysctl.conf

# 【重要】TCP全连接队列参数设置， 这样设置的目的是防止节点数较多（比如超过100）的ES集群中，节点异常重启时全连接队列在启动瞬间打满，造成节点hang住，整个集群响应迟滞的情况
echo "net.ipv4.tcp_abort_on_overflow = 1" >>/etc/sysctl.conf
echo "net.core.somaxconn = 2048" >>/etc/sysctl.conf

# 【重要】降低tcp alive time，防止无效链接占用链接数
echo 300 >/proc/sys/net/ipv4/tcp_keepalive_time
```

## ES节点配置

### 1. jvm.options

-Xms和-Xmx设置为相同的值，推荐设置为机器内存的一半左右，剩余一半留给系统cache使用。

- jvm内存建议不要低于2G，否则有可能因为内存不足导致ES无法正常启动或OOM
- jvm建议不要超过32G，否则jvm会禁用内存对象指针压缩技术，造成内存浪费

### 2. 【重要】elasticsearch.yml

- 设置内存熔断参数，防止写入或查询压力过高导致OOM，具体数值可根据使用场景调整。

``` 
indices.breaker.total.limit: 30% indices.breaker.request.limit: 6% indices.breaker.fielddata.limit: 3%
```

- 调小查询使用的cache，避免cache占用过多的jvm内存，具体数值可根据使用场景调整。
```
indices.queries.cache.count: 500
indices.queries.cache.size: 5%
```

- 单机多节点时，主从shard分配以ip为依据，分配到不同的机器上，避免单机挂掉导致数据丢失。 
```
cluster.routing.allocation.awareness.attributes: ip node.attr.ip: 1.1.1.1
```

## 【重要】控制index、shard总数量

上面提到，ES的元信息由master节点管理，定期同步给各个节点，也就是每个节点都会存储一份。这个元信息主要存储在cluster_state中，如所有node元信息（indices、节点各种统计参数）、所有index/shard的元信息（mapping, location, size）、元数据ingest等。

ES在创建新分片时，要根据现有的分片分布情况指定分片分配策略，从而使各个节点上的分片数基本一致，此过程中就需要深入遍历cluster_state。当集群中的index/shard过多时，cluster_state结构会变得过于复杂，导致遍历cluster_state效率低下，集群响应迟滞。腾讯云CES技术团队曾经在一个20个节点的集群里，创建了4w+个shard，导致新建一个index需要60s+才能完成。

当index/shard数量过多时，可以考虑从以下几方面改进：

- 降低数据量较小的index的shard数量
- 把一些有关联的index合并成一个index
- 数据按某个维度做拆分，写入多个集群

## 【重要】Segment Memory优化

前面提到，ES底层采用Lucene做存储，而Lucene的一个index又由若干segment组成，每个segment都会建立自己的倒排索引用于数据查询。Lucene为了加速查询，为每个segment的倒排做了一层前缀索引，这个索引在Lucene4.0以后采用的数据结构是FST (Finite State Transducer)。Lucene加载segment的时候将其全量装载到内存中，加快查询速度。这部分内存被称为SegmentMemory，`常驻内存，占用heap，无法被GC`。

前面提到，为利用JVM的对象指针压缩技术来节约内存，通常建议JVM内存分配不要超过32G。当集群的数据量过大时，SegmentMemory会吃掉大量的堆内存，而JVM内存空间又有限，此时就需要想办法降低SegmentMemory的使用量了，常用方法有下面几个：

- 定期删除不使用的index
- 对于不常访问的index，可以通过close接口将其关闭，用到时再打开
- 通过force_merge接口强制合并segment，降低segment数量

腾讯云CES技术团队在此基础上，对FST部分进行了优化，释放高达40%的Segment Memory内存空间。