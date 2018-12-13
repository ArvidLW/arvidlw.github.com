# elasticsearch 故障处理与分片恢复[持续更新]

# 符号与标记约定

## ??? 知识探索

## QQ 其它问题 

# 性能与故障测试工具

## 批量写入工具QQ

- logstash

## 批量查询工具QQ

- 脚本，模拟大规模查询
- 模拟用户刁钻搜索

## 故障模拟QQ

- 模拟故障，kill应用与节点

# elasticsearch 常用命令

## 查看分片是否分配

```bash
ip：9200/_cat/shards?v
```

# 节点假死

[扩容故障与节点假死]: https://www.jianshu.com/p/329b9f92ac4c

## 问题呈现

- 新加入的节点全部处于假死状态。已经有大部分数据同步到新节点，而且服务器已无法连接。

- 数据无法写入相应分片

## 寻找原因

ip:9200/_cat/node?v

显示信息中，节点没有状态信息，只有ip和name

查看日志发现关键字：

- FileSystemException
- /opt/elasticsearch/node6/data/nodes/0/indices/NP2tORUfSq6jl0lb5CzOVw/2/translog/translog.ckp: Too many open files in system

 也就是说同时打开的文件数达到了系统的限制，这也就是无法登陆系统的原因。
 不难理解上述问题的出现：一个服务器中配置了两个节点，这两个节点都运行在elastic用户下，该用户所在系统的limit.conf中对该用户同时打开的文件数有限制。而在集群同步数据的过程中，系统在大量的写文件，同时实时数据又在大量写入。这样就导致文件达到最大的阈值。因此导致elasticsearch假死。

## 解决办法

### 临时应急

```bash
# reroute不可用的索引
POST /_cluster/reroute
  {
  "commands": [
    {
      "allocate_stale_primary": {
        "index": "applog-prod-2017.08.16",
        "shard": 2,
        "node": "node7",
        "accept_data_loss" : true
      }
    }
  ]
}
# 需要注意的是，对于主分片执行reroute，一定需要所分配的节点上存在该索引的文件，否则会提示找不到索引
```

### 根本永久

- 修改用户limit.conf同时打开的文件数限制，退出bash再登录可以及时生效

```shell
# /etc/security/limits.conf
# 根据具体情况修改
user soft memlock unlimited
user soft memlock unlimited
user soft nofile 655360
user hard nofile 655360
user soft nproc 655360
user hard nproc 655360
```

- 重启相应节点

# 警惕通配符查询

[警惕通配符查询]: https://www.cnblogs.com/raoshaoquan/p/7083844.html

## 问题呈现

cpu平时利用在10%左右，无故飙高到100%，持续20分钟，导致大量用户搜索请求无响应。

## 寻找原因

使用(Wildcard Query)来实现模糊查询，搜索句子前后加上通配符\*，例如 \*迪士尼\* 来搜索含有迪士尼的相关产品。然而用户输入字符串的长度没有限制，用户一般直接复制并搜索，导致wild query查询很慢，非常消耗cpu。

## 解决办法

总结： wildcard query应杜绝使用通配符打头，实在不得已要这么做，就一定需要限制用户输入的字符串长度。 最好换一种实现方式，通过在index time做文章，选用合适的分词器，比如nGram tokenizer预处理数据，然后使用更廉价的term query来实现同等的模糊搜索功能。 对于部分输入即提示的应用场景，可以考虑优先使用completion suggester, phrase/term suggeter一类性能更好,模糊程度略差的方式查询，待suggester没有匹配结果的时候，再fall back到更模糊但性能较差的wildcard, regex, fuzzy一类的查询。

补记： 有同学问regex, fuzzy query是否有同样的问题，答案是有，原因在于他们底层和wildcard一样，都是通过将pattern构造成DFA来加速字符串匹配速度的。

## 问题复现

复现方法

1. 创建一个只有一条文档的索引

```
POST test_index/type1/?refresh=true
{
  "foo": "bar"
}
```

2. 使用wildcard query执行一个首尾带有通配符*的长字符串查询

```
POST /test_index/_search
{
  "query": {
    "wildcard": {
      "foo": {
        "value": "*在迪士尼乐园，点亮心中奇梦。它是一个充满创造力、冒险精神与无穷精彩的快地。您可在此游览全球最大的迪士尼城堡——奇幻童话城堡，探索别具一格又令人难忘的六大主题园区——米奇大街、奇想花园、梦幻世界、探险岛、宝藏湾和明日世界，和米奇朋友在一起，感觉欢乐时光开业于2016年上海国际旅游度假区秀沿路亚朵酒店位于上海市浦东新区沪南公路（沪南公路与秀沿路交汇处），临近周浦万达广场、地铁11号线秀沿路站，距离上海南站、人民广场约20公里，距离迪线距*"
      }
    }
  }
}
```

3. 查看结果

```
{
  "took": 3445,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": 
  }
}
```

即使no hits，耗时却是惊人的3.4秒 (测试机是macbook pro, i7 CPU)，并且执行过程中，CPU有一个很高的尖峰。

线上的查询比我这个范例要复杂得多，会同时查几个字段，实际测试下来，一个查询可能会执行十几秒钟。 在有比较多长字符串查询的时候，集群可能就DOS了。

探查深层次根源
为什么对只有一条数据的索引做这个查询开销这么高？ 直觉上应该是瞬间返回结果才对!

回答这个问题前，可以再做个测试，如果继续加大查询字符串的长度，到了一定长度后，ES直接抛异常了，服务器ES里异常给出的cause如下:

```bash
Caused by: org.apache.lucene.util.automaton.TooComplexToDeterminizeException: Determinizing automaton with 22082 states and 34182 transitions would result in more than 10000 states. at org.apache.lucene.util.automaton.Operations.determinize(Operations.java:741) ~[lucene-core-6.4.1.jar:6.4.1
```


该异常来自org.apache.lucene.util.automaton这个包，异常原因的字面含义是说*“自动机过于复杂而无法确定状态： 由于状态和转换太多，确定一个自动机需要生成的状态超过10000个上限"*

网上查找了大量资料后，终于搞清楚了问题的来龙去脉。为了加速通配符和正则表达式的匹配速度，Lucene4.0开始会将输入的字符串模式构建成一个DFA (Deterministic Finite Automaton)，带有通配符的pattern构造出来的DFA可能会很复杂，开销很大。这个链接的博客[using-dfa-for-wildcard-matching-problem](http://wpc.github.io/algorithm/2016/01/13/using-dfa-for-wildcard-matching-problem.html)比较形象的介绍了如何为一个带有通配符的pattern构建DFA。借用博客里的范例，a*bc构造出来的DFA如下图:

![DFA](https://elasticsearch.cn/uploads/article/20170511/e72c12e07d253a3c9d1413403c902d51.png)

## 相关理论 ???

[NFA与DFA]: http://www.cnblogs.com/Revenent-Blog/p/7908755.html	"确定性有穷状态机，不确定性无穷状态机"

粗浅的理解是: NFA在输入一个条件的情况下，可以从一个状态转移到多种状态，而DFA只会有一个确定的状态可以转移，因此DFA在字符串匹配时速度更快。 DFA虽然搜索的时候快，但是构造方面的时间复杂度可能比较高，特别是带有首部通配符+长字符串的时候。

Elasticsearch官方文档里对于wildcard query有特别说明，要避免使用通配符开头的term。

" Note that this query can be slow, as it needs to iterate over many terms. In order to prevent extremely slow wildcard queries, a wildcard term should not start with one of the wildcards * or ?."

# master堆内存溢出

[ttl导致OOM]: https://blog.csdn.net/daydayupzzc/article/details/73554800?utm_source=blogxgwz0
[2.3.3 ttl]: https://www.elastic.co/guide/en/elasticsearch/reference/2.3/mapping-ttl-field.html#_default_ttl



## 问题呈现

版本2.3.3，堆内存频繁溢出，重新启动后10几分钟，内存爆满，多次重启，重复OOM。

## 寻找原因

- 文档自定义_id太长，100个字符，使得搜索与分片迁移内存占用过高，
- ttl开启，且默认60s计算一次失效文档，PurgerThread周期任务占用资源，当周期短而文档多使得一直在计算失效文档时间戳。

## 解决办法

- 将indices.ttl.interval 设置成很大，比如 315360000s，约10年时间
- 禁用掉ttl清理
```
PUT index1/_settings
{
    "indices.ttl.interval": 315360000s
}

PUT index1/_settings
{
    "ttl.disable_purge": true
}
```

## 问题复现

## 相关理论

### 2.3.3 ttl设置参数

- indices.ttl.interval 默认 60s
- indices.ttl.bulk_size 默认 10000
- detect_noop 默认true

# 分片丢失恢复

[健康红色解决方案]: https://blog.csdn.net/laoyang360/article/details/78443006
[reroute]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/cluster-reroute.html

## 数据无法恢复，恢复分片

```
# 注意accept_data_loss为true时，才执行，表示接受数据丢失。一般用于节点数据丢失无法恢复时。当设置后，如果原来分片的节点重新加入，将会清空该节点之前的数据。
# allocate_empty_primary分配空分片
POST /_cluster/reroute
{
    "commands" : [
        {
          "allocate_empty_primary": {
            "index": "logstash-app-2018.11.26",
            "shard": 0,
            "node" : "data-55.15.252.146--hdd-11-0",
            "accept_data_loss": true
          }
        }
    ]
}
```



# 参数答疑

- cluster.routing.rebalance.enable

- cluster.routing.allocation.enable 为false时不能rebalance, 只能手动reroute移动分片

- dry run 只运行计算应用改变的结果，而不实际做出改变

  [reroute]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/cluster-reroute.html

- ?explain URI query, 解释为什么这个命令能或不能执行，支持如下

  - move
  - cancel
  - allocate_replica





# es 一直未分配分片

[集群indices 分片状态INITIALIZING]: https://cloud.tencent.com/info/173f5cf64a418e6382e779efc22e3d3d.html

 n unassign shards initializing shards

一直在初始化分配，一直在分配分片

解决步骤

1、查看在分配的分片索引是否非常大而导致初始化很久，是则等待

2、如果是很小的索引，初始化分片卡住了，那么，关闭自动relocate，然后重启相应节点

3、如果是有副本索引，其主分片正常，那么可以先将副本设为0，再设为1。

