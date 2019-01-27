# elasticsearch堆内存满cpu满及负载高问题探究

[滴滴es多集群管理]: https://mp.weixin.qq.com/s/K44-L0rclaIM40hma55pPQ

[logstash与ingest]: https://sematext.com/blog/elasticsearch-ingest-node-vs-logstash-performance/
[小故事]: https://mp.weixin.qq.com/s/Rs2ACXlrLHs5nVKwMwWOYA
[es性能优化]: https://elasticsearch.cn/article/6202

辩证思想

- 在同一需求下，改变实现方式，测试硬件、cpu、内存、网络、磁盘io的利用率占比。
- 注意不是打满，而是在一个正常工作的条件下，看不同情况下，瓶颈在哪里。

# logstash与ingest

ingest 过滤与字段解析，耗cpu，速度比logstash快10倍

logstash 过滤与字段解析，速度慢，规则多时，速度影响不大。

## **Performance Conclusions:** **Logstash vs Elasticsearch Ingest Node**

- Logstash is easier to configure, at least for now, and performance didn’t deteriorate as much when adding rules
- Ingest node is lighter across the board. For a single grok rule, it was about 10x faster than Logstash
- Ingest nodes can also act as “client” nodes
- Define the grok rules matching most logs first because both Ingest and Logstash exit the chain on the first match by default

## **Ingest Logstash performance tuning:**

- Make sure Logstash’s pipeline batch size and number of threads are configured to make the best use of your hardware: use all the CPU, but don’t spend too much time on context switching
- If parsing is simple, [Logstash’s Dissect filter](https://www.elastic.co/guide/en/logstash/current/plugins-filters-dissect.html) might be a good replacement for Grok
- For Ingest, it’s best to have dedicated Ingest nodes. By default, all nodes can perform Ingest tasks (node.ingest=true in elasticsearch.yml). Ingest work is almost all CPU, so you can choose compute-optimized machines for the job






