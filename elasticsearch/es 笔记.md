es 笔记

[ebay es优化]: https://yq.aliyun.com/ziliao/366800

index.routing_partition_size

随着副本数量增加吞吐量降低，同时响应时间变长。 尽量使用自生成ID。 Elasticsearch自生成ID是可用保证唯一的以避免版本查询。如果客户真的需要自定义ID，建议选一个对Lucene友好的ID，例如 zero-padded顺序ID、UUID-1和Nano时间。

