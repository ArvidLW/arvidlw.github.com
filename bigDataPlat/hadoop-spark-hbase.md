# hadoop-spark-hbase等笔记

***

记录应用中出现的问题与学习记录

# 读取hbase

***

## spark读取hbase

1. sparksql
2. sparkconf

## HBase客户端

1. HBaseConfiguration

# HBase shell

***

## 增加列族

alter 'tablename', {NAME=>'columnName', VERSION=>5}

