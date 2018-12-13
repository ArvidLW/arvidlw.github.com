# Es备份恢复策略研究与实现

[snapshots]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/modules-snapshots.html

# 备份方式

es备份

- 插件备份
  - hdfs备份
  - s3备份
- fs备份

hdfs备份

- 备份到s3



# es备份

## API

查看正在运行的备份
```
GET /_snapshot/my_backup/_current
```
查看全部备份
```
GET /_snapshot/my_backup/_all
```

查看相应snapshot情况, 包含备份时长与备份的状态

```
GET /_snapshot/my_backup/mysnap
```



## 特性

增量备份

## 缺点

不能同时备份多个索引

## 备份到hdfs

配置： es物理机，hdfs虚拟机

索引大小：10g

耗时间：9min

结果：1g/min

恢复：1.4min主分片，2.7min副本，全部4.1min

可能瓶颈：

- hdfs写入速度不够快，因为恢复时间小于写时间

解决方式

- 测试hdfs写入性能