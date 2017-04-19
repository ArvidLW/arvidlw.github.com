# 面试Q&A

***

# 基础问题

***



## unix内核结构

### b+树

### tcp三次握手

### 网络编程

### i/o复用

### mysql性能优化，索引

#### 选择索引数据类型

1、越小的数据类型通常更好：越小的数据类型占用空间少，处理速度快

2、简单的数据类型

3、避免用NULL：应该指定列为NOT NULL，在MYSQL中含有空值的列很难查询优化，因为他们使得索引和索引的统计信息以及比较运算更加复杂。应该用0或特殊的值或一个空串代替空值。

#### 选择标识符

整型：处理快，作为标识符最好

字符串：尽量避免使用，处理慢，消耗空间，由于字符串为随机的，所以在索引中的位置也是随机的，会导致页面分裂、随机访问磁盘，聚簇索引分裂。

#### 索引入门

组合索引：列的顺序非常重要，mysql仅能对索引最左边的前缀进行有效的查找。

#### 聚簇索引

一个表只能有一个聚簇索引，表中记录的顺序与索引中的顺序相对应，查询比非聚簇索引快。对于频繁更新的列，不适合聚簇索引，因为更新索引表时还要移动索引数据物理位置。

#### 非聚簇索引



### 线程与进程

### 数组、链表、c++与java区别

### linux命令

#### 查看内存

查看硬件信息：dmidecode, 其中dmi为desktop manage interface

-t 或 --type为类型，关键字：bios,system,baseboard,chassis,processor,memory,cache,connector,slot

查看内存信息：dmidecode -t memory



查看内存使用情况

top 查看cpu使用状况，并会定时更新，退出top用q键

可以选择按进程或者按用户查看

按用户：top -u lw

free 查看未使用内存的情况，比 top开销小

free -m

​		   total       used       free     shared    buffers     cached
Mem:           976        475        500          0         13        101
-/+ buffers/cache:        360        615
Swap:         1021        209        812

结果中mem为系统内存，对系统而言，总共976，用了475，还有500，其中475的还有buffers 13，和cahced 101没有用

-/+ buffers/cache是对应用程序而言，已用的为360，还有615可以用。

360=475-13-101

615=500+13+101

buffers为从内存将要写到磁盘的内容，或其它I/O的内容

cache是从磁盘或其它I/O读进来的数据临时存放区

#### 查看网络状态

查看当前监听80端口的程序

lsof -i:80

查看路由信息

netstat -rn

查看端口使用情况，即listen的端口

netstat -ln

netstat -tln 注意 -t为查看tcp, -u为查看udp, -a显示所有socket

# 项目问题

***

## HBASE

逻辑模型

物理模型

1、table中所有行都按照row key字典序排列

2、Table在**行的方向**上分割为多个Region

3、每个表开始只有一个region，随着数据增多会自动进行分割成多个。

4、Region是Hbase中**分布式存储和负载均衡最小**的单元，不同Region分布到不同RegionServer上

5、Region不是存储的最小单元，Region由一个或者多个store组成，每个store保存一个Columns Family，每个Store又由一个memStore和0-多个StoreFile组成，StoreFile包含HFile；memStore存储在内存中，StoreFile存储在HDFS中。

## HBASE基本组件

### client

包含访问HBASE的接口，并维护cache来加快对HBASE的访问，比如region的位置信息

### Master

为Region Server分配region

负责Region Server负载均衡

发现失效的Region Server并重新分配其上的region

管理用户对table的增删改查操作

### Region Server

维护region,处理region的IO

负责切分在运行过程中变得过大的region

### Zookeeper

通过选举，保证有一个Master

存储所有Region的寻址入口

实时监控Region server的上线和下线消息并通知给Master

存储HBase的schema和table元数据

默认情况下，HBase管理zookeeper实例

Zookeeper的引入使得Master不再有单点故障