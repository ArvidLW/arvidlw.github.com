# es命令错误与原因

## 未完全安装hdfs 插件错误

[repository verification exception]: https://discuss.elastic.co/t/getting-repository-verification-exception-while-creating-snapshot/81945
[snapshot repository error]: https://discuss.elastic.co/t/elasticsearch-create-a-snapshot-repository-error/60830

新建snapshot仓库失败。

```
put _snapshot/hdfs198
{
    "type":"hdfs",
    "settings":{
        "uri": "hdfs//ip:9000",
        "path": "path"
    }
}

error_cause
type: repository_verification_exception
RemoteTransportException
RepositoryMissingException[[hdfs198] missing

```

