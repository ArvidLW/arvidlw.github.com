# es 备份恢复

配置

es-2.X的属性

```
security.manager.enabled: false
```

es备份

- 备份hdfs
- 备份ecs
- 备份fs

[backup]: https://www.elastic.co/guide/en/elasticsearch/plugins/6.5/repository.html
[snapshots]: https://www.elastic.co/guide/en/elasticsearch/reference/6.5/modules-snapshots.html
[snapshot 6.1]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/modules-snapshots.html



[repository]: https://www.elastic.co/guide/en/elasticsearch/plugins/6.1/repository.html
[snapshots]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/modules-snapshots.html

# 问题

配置目录，数据节点不能同master共用目录

而且多实例data共用目录，这样不行。

