# es索引原理对比

参考：beaver.pdf

## 多segment

单进程，提高并发度能利用多核能力。

doc分布，doc分布在不同的segment提高分布能力。

### 知晓

- 数据结构和算法限制DOC必须按照DOCID的顺序写入
- Lucene和Beaver在将Doc添加到索引结构时都必须是单线程
- Lucene一个Segment只能由同一个线程同步写入

## Flush

- 内存大小有限
- 词典、列存保存存的字符串没有顺序保证，查找慢
- 数据结构不够紧凑
- Flush时Segment只读，DOC集确定
  - 重新整理倒排表，列存提高查找效率
  - 根据数据特征对数据进行压缩
- 磁盘中，所有项目都会分为两个文件
  - Meta文件：存领储索引信息，文件小(.dvm,.fdx,.tip, ...)
  - 数据文件：存储具体数据，文件大(.dvd, .fdt, .tim, ...)

## merge

- 更少的索引文件数
- 更多的压缩机会
- 更高的I/O利用率
- 更贴合的数据结构

## shard

特点

- 不可拆分
- 大小和数目没法直接确定

shard概念引入后

- 可以由任意多少Segment组成
- 可以随时增加/减少Segment
- Segment之间可以进行合并操作
- 搜索统计都是以Shard为基本单位

## Replica

WAL(Write-Ahead-Logging)

写之前记录日志

- Translog记录所有请求入库的文档
- Translog写两份就可以认为完成备份
- MemBlock可以慢慢写入

## Recovery

机器挂/进程挂

- 内存数据丢失
- 从Translog恢复内存数据

重新备份

- 重新建立Primary-Backup关系
- 尽量利用现有数据而不是重新拷贝

## 总结一下translog的功能

1.保证在filesystem cache中的数据不会因为elasticsearch重启或是发生意外故障的时候丢失。

2.当系统重启时会从translog中恢复之前记录的操作。

3.当对elasticsearch进行CRUD操作的时候，会先到translog之中进行查找，因为tranlog之中保存的是最新的数据。

4.translog的清除时间时进行flush操作之后（将数据从filesystem cache刷入disk之中）。 

## 再总结一下flush操作的时间点：

1.es的各个shard会每个30分钟进行一次flush操作。

2.当translog的数据达到某个上限的时候会进行一次flush操作。 

## translog和flush的一些配置项：

index.translog.flush_threshold_ops:当发生多少次操作时进行一次flush。默认是 unlimited。

index.translog.flush_threshold_size:当translog的大小达到此值时会进行一次flush操作。默认是512mb。

index.translog.flush_threshold_period:在指定的时间间隔内如果没有进行flush操作，会进行一次强制flush操作。默认是30m。

index.translog.interval:多少时间间隔内会检查一次translog，来进行一次flush操作。es会随机的在这个值到这个值的2倍大小之间进行一次操作，默认是5s。

# 优化点

## 内存结构

### ES/Lucene

- 不可搜索&统计的内存结构(NRT)
- 优点
  - 内存结构紧凑，省空间
- 缺点
  - 准实时索引检索引擎
  - 模拟实时需要定期refresh将数据转化成磁盘结构(flush)
  - 所有检索都可能要耗费I/O

### Beaver

- 全功能搜索&统计的内存结构(RT)
- 优点
  - 内存原地检索不必耗费I/O
  - 纯实时索引检索引擎
  - Block更大
- 缺点
  - 内存结构稍松散需要耗费更多内存
  - 内存中词典没排序，通配检索词典消耗较大
  - 实现更加复杂

## 【重要】并发控制

### ES/Lucene

- segment只能由一个线程写
- 优点
  - 同步写代码逻辑简单
- 缺点
  - 往往需要大量的segment才能用满cpu。？这难道不是优点吗

### Beaver

- segment支持内部并发写
- 优点
  - 几个Block就能用满cpu。？效率是否高
- 缺点
  - 采用异步流水线实现，代码复杂
  - 结合内存数据结构可检索，实现难度大

## Global Ordinal管理

### ES/Lucene

- 静态Global Ordinal: 每次Segment变动都会导致Global orginal失效
- 优点
  - 所有的统计和排序都能完全通过数值比较得到
- 缺点
  - 实时系统中会导致频繁的Glogbal Ordinal失效和重建

### Beaver

- 动态Global Ordinal：如果有新增的字符串，则直接通过字符串比较
- 优点
  - Global ordinal能长期使用
- 缺点
  - 需要一部分内存保存新增的字符串
  - 基于字符串的比较效率低于数值比较

## 【重要】内存结构和并发的影响

### ES/Lucene

- 产生的Segment的数目极多，每个segment的Doc数目少，严重依赖Merge。
- Merge会占用大量的IO和CPU。
- Merge不及时会导致低性能。
- 由于Segment频繁变动导致Global Ordinal大量失效。

### Beaver

- 产和的Block少，仅在一些特殊的情况下会产生小Block。
- Merge的规模小，正常情况下因merge产生的IO和CPU消耗很低。
- Global ordinal构建一次后往往能使用很长时间。

## Meta管理与延时加载

### Es/Lucene

- 所有打开的Segment都需加加载全部数据的Meta
- 优点
  - 搜索时不用加载Meta数据
- 缺点
  - 能同时找开的索引数目有限制
  - Meta往往耗费大量的内存
  - Phantom Index需要额外的运维流程管理

### Beaver

- 选择性加载Meta, 并通过LRU的策略自动管理Meta cache
- 优点
  - 可以管理的Block没有内存限制
  - Meta耗费的内存可以控制
  - 全自动管理，不需要其他流程
- 缺点
  - 如果BLock Meta没有LRU Cache中则需要加载，可能耗费较长的时间

## 针对时间戳的优化

### ES/Lucene

- 无时间戳记录
- 优点
  - 不需要记录时间戳，Segment的meta信息稍小
- 缺点
  - 每次检索都需要检索全部segment
  - 按时间排序取Top N的算法不够高效

### Beaver

- 对每个Block都保存了时间戳范围
- 优点
  - 可以据据时间戳和Query的范围排除大量不需要检索的block
  - 某些情况下可以优化query,不进行时间戳过滤
  - 按时间戳排序取Top N时排除掉大量的Block
- 缺点
  - 需要维护时间戳信息

## 复制控制

### ES/Lucene

- 双写
- 优点
  - 恢复速度快，Replica能迅速提升
- 缺点
  - replica的索引也需要耗费和primary相同的资源

### Beaver

- 数据拷贝
- 优点
  - Replica不需要消耗资源去索引
  - 2个副本能直接省一倍的资源
- 缺点
  - 耗费更鑫网络带宽
  - 只有primary能检索，replica提升有个recovery过程

## Mapping

### ES/Lucene

- 关键路径
- 优点
  - 每次都能够获取较新mapping
- 缺点
  - mapping更新为关键路径，有可能严重影响index过程

### Beaver

- 异步拷贝
- 优点
  - 后台线程更新Mapping不影响索引过程
- 缺点
  - 有一定几率获取不到最新的mapping

## 集群路由

### ES/Lucene

- UID路由：UID确定后Doc只能写到特定的shard
- 缺点：
  - UID确定后单个shard故障将影响集群入库（短板效应）

### Beaver

- UID 路由或随机路由
- 优点
  - 更灵活的路由策略，当不需要排重时可以选择随机路由
  - 随机路由无短板效应，单shard故障不影响入库
- 缺点
  - 随机路由不能排重
  - 当shard故障时可能导致这些shard的数据量减少。





