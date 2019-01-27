es集群内存与数据量比例以及性能测试

# 需求

背景：

数据迁移：5-10T

冷热分离： 1G/s?  1G/min

场景：

1、数据量达到80%，90%，时查询性能。

2、进行数据导入时，查询数据的性能。

结果：

1、哪种节点配比好。一个共多少个节点，一个节点几个实例。

2、给出使用es最佳方法，及相应性能达到多少

# 测试方法

## 环境准备

### 145-147

### 148-150

- 3个节点
- 实例：9hdd+2ssd
- client：1个
- 35个实例

### 154-156

## 数据准备

- 导入city数据，原数据大小1.5g.
- 导入日志数据大小，原数据大小100M.



## 测试过程

148 logstash test-ES2 30节点 topic szyw-error 283056/sec

节点 cpu load

148 74-96% 113.76-136%

149 22-35% 23-26.78%

150 22-37% 20.12-25%

# 测试第一阶段

# es新特性

rollup 定时功能，聚合某些索引以供查询。

shrink 收缩索引，将一个索引收缩成另一个，然后再别名切换索引。

基于官方curator



# es表现

1、当集群red后，其不会relocate

2、内存清理，java gc

3、es_node_name_suffix部署脚本设置



# 手动gc

[jstat]: http://www.cnblogs.com/lizhonghua34/p/7307139.html
[jstat2]: https://www.jianshu.com/p/213710fb9e40
[jstack]: https://blog.csdn.net/tzs_1041218129/article/details/61630981
[jmap]: https://www.cnblogs.com/kongzhongqijing/articles/3621163.html
[target process not responding or HotSpot VM not loaded]: https://www.cnblogs.com/yidiandhappy/p/7826383.html
[解决 no responding 切换到相应用户]: https://stackoverflow.com/questions/7420501/jstack-target-process-not-responding

jstat -class pid

- Loaded:加载class的数量
- Bytes：所占用空间大小
- Unloaded：未加载数量
- Bytes:未加载占用空间
- Time：时间

jstat -compiler pid

- Compiled：编译数量。
- Failed：失败数量
- Invalid：不可用数量
- Time：时间
- FailedType：失败类型
- FailedMethod：失败的方法

jmap -histo:live pid 这个命令执行，JVM会先触发gc，然后再统计信息。

在用cms gc的情况下，执行jmap -heap有些时候会导致进程变T，因此强烈建议别执行这个命令，如果想获取内存目前每个区域的使用状况，可通过jstat -gc或jstat -gccapacity来拿到。





# 第一阶段内存与磁盘配比

数据进行最大最小标准化后的比较。

纵轴：数据size大小34TB-154TB。

横轴：内存大小6.2-17.8

1、内存占用与数据大小相关性很强。

2、不考虑查询，约每Tb占用4g的内存，即1g内存对应约0.25TB数据。

即200TB数据，要800g内存。

3、考虑查询与数据导入，索引占内存不能超过60%。

即800/60%=1334,所以最小要1334g内存。故第TB占用4/0.6=6.7。

5、像生产集群34的data实例，1334/34=40g,则每个实例需要有40g的内存,才能支持200T数据。

6、segments合并前后，其占用内存差别不大，对于静态状态（没有导入与查询），基本没有影响。

小结：以上对于集群的规模与部署可以参考。由此得出：对于40T的物理机，不考虑cpu的使用，可设置实例数为40T*6.7/31=8.6个。考虑在备份、导入、查询等综合情况对cpu的使用，以及master,client也在物理节点的启用。最终得到实例data数为7个每个物理机。

6、这部分性能的调优，可以对目前我们所部署集群的数据进行分析得出精确值。

7、查询导入性能需要进一步测试。涉及segments合并前后的查询性能对比。

改进：

1.	静态配比多次验证。

2.	索引缓存大小调整是否改变静态配比。没加载的索引放在磁盘交换空间？

3.	查询如何影响性能。

需求：

磁盘利用80%、90%。

了解外部使用情况，如磁盘配比。

前海机子：

利用：85-94

工具：

部署脚本201



# 现象记录

当分片数过多时，client压力大

当分片较少时，master压力大。



# es优化

1、物理机部署脚本没有monit stop，没有stop 但是执行monit stop会不再对原应用监控。

2、stale原理，stale是将分片在原有节点的数据加载到节点上。

3、原数据问题，es master数据不同步。当磁盘快满或者负载大，导致元数据不同步？

4、fullgc, jmap -histo:live pid

5、分片无法恢复可能？关索引，删索引；关索引，开索引。

6、最小的master设置为2比较合理





# 物理机与虚拟机比较

## 维度

硬件资源

## 测试度量

磁盘io

网络



## 可能改进

多实例配置，共享文件夹，减少多于io？



多实例设置实例线程

https://www.elastic.co/guide/en/elasticsearch/reference/6.5/modules-threadpool.html





###  Processors setting[edit](https://github.com/elastic/elasticsearch/edit/6.5/docs/reference/modules/threadpool.asciidoc)

The number of processors is automatically detected, and the thread pool settings are automatically set based on it. In some cases it can be useful to override the number of detected processors. This can be done by explicitly setting the `processors` setting.

```yaml
processors: 2
```

There are a few use-cases for explicitly overriding the `processors` setting:

1. If you are running multiple instances of Elasticsearch on the same host but want Elasticsearch to size its thread pools as if it only has a fraction of the CPU, you should override the `processors` setting to the desired fraction (e.g., if you’re running two instances of Elasticsearch on a 16-core machine, set `processors` to 8). Note that this is an expert-level use-case and there’s a lot more involved than just setting the `processors` setting as there are other considerations like changing the number of garbage collector threads, pinning processes to cores, etc.
2. Sometimes the number of processors is wrongly detected and in such cases explicitly setting the `processors` setting will workaround such issues.

In order to check the number of processors detected, use the nodes info API with the `os` flag.





https://cloud.tencent.com/developer/article/1370560














