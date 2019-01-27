# elasticsearch-6.1.4 + kibana x-pack部署

[x-pack安装]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/installing-xpack-es.html
[no monitoring data found]: https://discuss.elastic.co/t/no-monitoring-data-found-indexnotfoundexception/155431
[monitoring settings]: https://www.elastic.co/guide/en/elasticsearch/reference/5.5/monitoring-settings.html
[xpack monitoring settings]: https://www.elastic.co/guide/en/x-pack/5.5/monitoring-cluster.html#http-exporter-reference
[效果]: https://www.elastic.co/guide/en/kibana/6.1/elasticsearch-metrics.html
[interactive]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/installing-xpack-es.html
[kibana安装xpack]: https://www.elastic.co/guide/en/kibana/6.1/installing-xpack-kb.html
[auto|interactive]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/setup-passwords.html



## ingest开启

elasticsearch 节点的ingest设置是对数据进行过滤的功能。当ingest为true时，可以定义pipline对录入的数据进行过滤，增加字段，格式化数据等。类似于logstash。节点默认node.ingest:true

记得一篇文章对比了ingest,logstash。表明ingest是logstash速度的10倍。但是ingest内存消耗大，而logstash比较稳定。

在安装x-pack时，需要开启节点ingest功能。可以设置client节点node.ingest: true。

## x-pack安装

最好以普通用户权限安装，避免es用普通权限运行，出现无权限情况。

```bash
# 对于es
su {{es_user}} yes | bin/elasticsearch-plugin install file:///path/to/file/x-pack-6.1.4.zip
# 对于kibana(同一个x-pack包)
su {{es_user}} yes | bin/kibana-plugin install file:///path/to/file/x-pack-6.1.4.zip
# 启动es
./bin/elasticsearch
# 配置密码(只能初始配置一次)
bin/x-pack/setup-passwords interactive # 手工输入
# 12345677 | bin/x-pack/setup-passwords interactive # 手工输入
# 或者
bin/x-pack/setup-passwords auto # 随机生成密码
# 配置kibana.yml
elasticsearch.username: "kibana"
elasticsearch.password: "kibanapassword"
# 问题
no monitoring data found
# 解决
配置 elasticsearch
# 配置elasticsearch.yml
xpack.monitoring.exporters:
  my_remote:
    type: http
    host: ["http://es-mon-1:9200", "http://es-mon2:9200"] 
    auth.username: remote_monitor 
    auth.password: changeme
    # index.name.time_format: YYYY-MM
# 重启elasticsearch 好了
```

## 监控设置elasticsearch.yml

[config索引]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/config-monitoring-indices.html
[监控设置]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/monitoring-settings.html
[x-pack特性设置]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/settings-xpack.html
[启动检查]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/bootstrap-checks.html

## X-PACK license

[settings]: https://www.elastic.co/guide/en/kibana/6.1/settings-xpack-kb.html



[x-pack]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/license-settings.html



## 注意事项

1、当配置密码后重启什么的，要隔一段时间再能访问，需要等。开始访问会invalid user or password

2、注意node.ingest=true 否则kibana monitoring 查不到数据。

## 典型错误

1、There are no ingest nodes in this cluster, unable to forward request to an ingest node

[问题解决]: https://elasticsearch.cn/question/1915

没有开启node.ingest

2、org.elasticsearch.xpack.monitoring.exporter.exportexception failed to flush export bulks。

没有配置xpack.monitoring.exporters或者默认的配置type:local用不了(The `local` exporter is the default exporter used by Monitoring)。每个es节点都需要配置。elasticsearch.yml

[exporter]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/monitoring-settings.html


```
xpack.monitoring.exporters:
  my_remote:
    type: http
    host: ["http://es-mon-1:9200", "http://es-mon2:9200"] 
    auth.username: remote_monitor 
    auth.password: changeme
    index.name.time_format: YYYY-MM
```

[xpack.monitoring配置]: https://www.elastic.co/guide/en/x-pack/5.5/monitoring-cluster.html#http-exporter-reference

3、Unhandled rejection [export_exception] Exception when closing export bulk

没有开启node.ingest



# 状态

index.refresh_interval:5s

# ES查询测试

## 查询默认值

[query]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/search-request-body.html
[from  to size]: https://www.elastic.co/guide/en/elasticsearch/reference/6.1/search-request-from-size.html

查询默认返回条件size=10，如果是只想要查询匹配条数与聚合，那么可以设置size=0，可能会提高性能。

默认from=0

Note that `from` + `size` can not be more than the `index.max_result_window` index setting which defaults to 10,000

The `took` time in the response contains the milliseconds that this request took for processing, beginning quickly after the node received the query, up until all search related work is done and before the above JSON is returned to the client. This means it includes the time spent waiting in thread pools, executing a distributed search across the whole cluster and gathering all the results.



## 查询语法

1、index_name/_search, 默认返回size=10。

2、

```js
GET /_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
GET /twitter/tweet/_search
{
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
GET /_search
{
    "from" : 0, "size" : 10,
    "query" : {
        "term" : { "user" : "kimchy" }
    }
}
POST /twitter/tweet/_search?routing=kimchy
{
    "query": {
        "bool" : {
            "must" : {
                "query_string" : {
                    "query" : "some query string here"
                }
            },
            "filter" : {
                "term" : { "user" : "kimchy" }
            }
        }
    }
}
```

# ES预聚合

es job，job任务，定期执行任务。

# ES INGEST

过滤管道



# ES 性能

[性能优化]: https://toutiao.io/posts/lkqldx/preview
[pb级数据分库分表]: https://elasticsearch.cn/article/6171





# ES 备份恢复

1、master原数据信息丢失



# ES查询可能瓶颈

1、client cpu

2、多个client同时查

3、分片数









