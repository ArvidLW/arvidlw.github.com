# 分词插件

Es vesion 6.1.4

## 标准分词器

```
POST _analyze
{
  "analyzer": "standard",
  "text": "而官网下载真的太慢了，就在想着有没有办法做到离线下载，当然了选择第三个选项使用本地的也就完了"
}
结果：
{
  "tokens": [
    {
      "token": "而",
      "start_offset": 0,
      "end_offset": 1,
      "type": "<ideographic>",
      "position": 0
    },
    {
      "token": "官",
      "start_offset": 1,
      "end_offset": 2,
      "type": "<ideographic>",
      "position": 1
    }
  ]
}
```

## jieba

https://github.com/sing1ee/elasticsearch-jieba-plugin   

这个有 6.0.0 和 6.4.0 版本的。

对于 6.1.4，可以clone下来后改相应的依赖包和版本判断的语句，再进行编译打包就ok. 不然会出现版本不兼容的错误。

- 可以添加自己的词典

### 应用分词

建立索引时，设定分词方式与过滤器方式。

```
# 例1
# 统一设置过滤器与分词器
PUT jieba_index
{
  "settings": {
    "analysis": {
      "filter": {
        "jieba_stop": {
          "type":        "stop",
          "stopwords_path": "stopwords/stopwords.txt"
        }
      },
      "analyzer": {
        "my_ana": {
          "tokenizer": "jieba_index",
          "filter": [
            "lowercase",
            "jieba_stop"
          ]
        }
      }
    }
  }
}

# 例2
# 对不同字段设置不同分词和搜索方式
{
  "company_data_1130": {
    "aliases": {
      "company_data": {}
    },
    "mappings": {
      "doc": {
        "properties": {
          "branch": {
            "properties": {
              "company_name": {
                "type": "text",
                "analyzer": "jieba_index",
                "search_analyzer": "jieba_search"
              },
              "registered_address": {
                "type": "text",
                "analyzer": "jieba_index",
                "search_analyzer": "jieba_search"
              },
              "type": {
                "type": "text"
              }
            }
          },
          "city": {
            "type": "text",
            "analyzer": "jieba_index"
          },
          "company": {
            "type": "text",
            "analyzer": "jieba_index",
            "search_analyzer": "jieba_search"
          },
          "entityScore": {
            "type": "double"
          },
          "province": {
            "type": "keyword"
          },
          "query": {
            "properties": {
              "match": {
                "properties": {
                  "company": {
                    "type": "text"
                  }
                }
              }
            }
          }
        }
      }
    },
    "settings": {
      "index": {
        "creation_date": "1543549055273",
        "number_of_shards": "2",
        "number_of_replicas": "1",
        "uuid": "Y0fJnuUlSuaUKV-IFulpew",
        "version": {
          "created": "6010499"
        },
        "provided_name": "company_data_1130"
      }
    }
  }
}
```

### 效果

```
POST jieba_test/_analyze
{
  "analyzer": "my_ana",
  "text": "我很开心"
}
# 结果
{
  "tokens": [
    {
      "token": "很开心",
      "start_offset": 1,
      "end_offset": 4,
      "type": "word",
      "position": 1
    },
    {
      "token": "开心",
      "start_offset": 2,
      "end_offset": 4,
      "type": "word",
      "position": 2
    }
  ]
}
```

### 分词与查询准确度影响分析

positionIncrement: 分词后的词距表示，影响搜索结果。

[Elasticsearch分词中的PositionIncrement何解]: https://github.com/sing1ee/kotlin-road/blob/master/ES-analysis-positionincrement.md

## IK

## HANLPx

# 2.3.3jieba

[2.3.3]: https://github.com/huaban/elasticsearch-analysis-jieba

./bin/plugin install file:///home/vcap/elasticsearch-analysis-jieba-2.3.3-bin.zip

cp -r ./plugins/jieba/dic ./config/

```
curl -XPUT 127.0.0.1:9200/test -d '{
    "settings" : {
        "number_of_shards" : 1,
        "number_of_replicas" : 0

    },
    "mappings" : {
        "test" : {
            "_all" : { "enabled" : false },
            "properties" : {
                "name" : { "type" : "string", "analyzer" : "jieba_index", "search_analyzer" : "jieba_search" }
            }
        }
    }
}';echo



curl 'http://127.0.0.1:9200/test/_analyze?analyzer=jieba_index' -d '中华人民共和国';echo
curl 'http://127.0.0.1:9200/test/_analyze?analyzer=jieba_search' -d '中华人民共和国';echo
curl 'http://127.0.0.1:9200/test/_analyze?analyzer=jieba_other' -d '中华人民共和国 HelLo';echo
```

