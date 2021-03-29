#  Elasticsearch随笔

##  python如何使用elasticsearch

```
from elasticsearch import Elasticsearch
实例化一个对象
es = Elasticsearch()
```

+ 插入数据

```
doc = {
    'author': 'author_name',
    'text': 'Interensting content...',
    'timestamp': datetime.now(),
}
```

```
插入数据 res = es.index(index="test-index", id=1, body=doc)
```

+ 查看数据状态

```
ipython
res['result']
目前我看见了 created updated等等
```

+ 如何查询数据

```
res = es.search(index="test-index", body={"query": {"match_all": {}}})
直接使用index也可以
```

+ 直接删除数据

```
es.delete(index="test-index", id=1)
```

## Requests连接ES

+ 简单增加

```python
url ='http://localhost:9200/my-index-000004/_create/3'
data ={
    '@timestamp': '2099-11-15T13:12:00',
     'message': 'GET /search HTTP/1.1 200 1070000',
     'user': {'id': 'kimchy'}
 	}
response = requests.post(url=url,json=data)
response.json()获取结果
```

+ 简单删除

```python
url = 'http://localhost:9200/my-index-000004/_doc/1?pretty'

response = requests.delete(url=url)
```

+ 简单修改

```shell
url = 'http://localhost:9200/my-index-000004/_update/2'

data = {'script': "ctx._source.new_field = 'value_of_new_field'"}

response = requests.post(url=url,json=data)
```

+ 简单查找

```shell
url = "http://localhost:9200/my-index-000004/_doc/2"

response = resquests.get(url=url)
```





##  不用Python连接es

###  我的笔记

```
linux直接访问es
	curl -X GET "localhost:9200/_cat/health?v=true&pretty"
或者
	curl -X GET "HTTP://localhost:9200/_cat/health?v=true&pretty"
这两个一样
```

Elasticsearch有多种摄取选项，但最终它们都做同样的事情：将JSON文档放入Elasticsearch索引中。

####  简单的数据存入

```json
curl -X PUT "localhost:9200/customer/_doc/1?pretty" -H 'Content-Type: application/json'    -d
							--索引         id                  类型
'
{
  "name": "John Doe"
}
'
```

这样就会把数据存入es数据库，索引是customer，id是1 ，内容就是-d后面的字典内容

**对应上面的数据查询**

```
curl -X GET "localhost:9200/customer/_doc/1?pretty"
后面参数带上pretty,返回的JSON数据就会变得好看格式
```

**Linux中获取es数据库所有索引内容**

```
curl "localhost:9200/_cat/indices?v=true"
后面的参数  v=true 会显示字段头
```

**Linux中批量提交JSON数据到es数据库**

```
curl -H "Content-Type: application/json" -XPOST "localhost:9200/bank/_bulk?pretty&refresh" --data-binary "@accounts.json"
大量的提交
```



#### Linux查询对应索引的前十条数据

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} }, # 全部匹配
  "sort": [                     # 排序
    { "account_number": "asc" } # 升序
  ]
}
'
```

+ 随时调用

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} }
}
'
```



#### Linux自定义查询任意条数据

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}
'
```

**以下请求搜索该`address`字段以查找地址包含`mill`或的客户`lane`：**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "address": "mill lane" } }
}
'
还是默认是10条数据
```

**同上，但是这次是查找词组**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
'
即address字段必须包含"mail lane"
不区分大小写
```

**使用更多更复杂的查询需要使用bool字段**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "age": "40" } }
      ],
      "must_not": [
        { "match": { "state": "ID" } }
      ]
    }
  }
}
'
上述表达式查找年龄40而且state不等于ID
默认也是10条
布尔查询中的每个must，should和must_not，filter元素称为查询子句。
```

**范围过滤器，它是在bool字段之下的过滤**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000
          }
        }
      }
    }
  }
}
'
```

**汇总分析结果**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {    # 表达式        
        "field": "state.keyword"
      }
    }
  }
}
'
上述表达式表明了汇总的字段是state
```

**跟上述表达式一样，但在其中加入了新的字段**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'

```

**这是上两个表达式的集合**

```shell
curl -X GET "localhost:9200/bank/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword",
        "order": {
          "average_balance": "desc"
        }
      },
      "aggs": {
        "average_balance": {
          "avg": {
            "field": "balance"
          }
        }
      }
    }
  }
}
'
表示以state为一组，降序排列，还有平均balance
```



#  REST API

##  文档api

###  CREATE API(增加单一索引)

+ 第一种增加方法

```shell
curl -X PUT "localhost:9200/my-index-000001/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
'
```

+ 第二种增加方法

```shell
curl -X PUT "localhost:9200/my-index-000001/_doc/1?op_type=create&pretty" -H 'Content-Type: application/json' -d'
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
'
```

+ 第三种增加方法

```shell
curl -X PUT "localhost:9200/my-index-000001/_create/1?pretty" -H 'Content-Type: application/json' -d'
{
  "@timestamp": "2099-11-15T13:12:00",
  "message": "GET /search HTTP/1.1 200 1070000",
  "user": {
    "id": "kimchy"
  }
}
'
```

**第一四种方法如何重复都不会报错，而第二三四中如果存在索引会报错**

**每一个方法只能有唯一的type，目前不能改变**

+ 第四种增加方法

```shell
curl -X PUT "localhost:9200/my-index-000003/_doc/1?pretty" -H 'Content-Type: application/json'    -d'
{
  "name": "John Doe"
}
'
```



****

###  DELETE API(删除一个索引值)

根据索引和_id来删除一个数据

+ 路由删除

  ```
  curl -X DELETE "localhost:9200/my-index-000001/_doc/1?routing=shard-1&pretty"
  ```

+ Timeout删除(超时报错) 

    ```
  curl -X DELETE "localhost:9200/my-index-000001/_doc/1?timeout=5m&pretty"
  ```

+ 普通删除

  ```
  curl -X DELETE "localhost:9200/my-index-000004/_doc/1?pretty"  
  ```





###  DELETE BY QUERY(查询并删除) 

+ 简单示例

```shell
curl -X POST "localhost:9200/my-index-000001/_delete_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "user.id": "elkbee"
    }
  }
}
'
删除所有user.id是"elkbee"的人
```

+ 删除单一索引内的全部内容

```shell
curl -X POST "localhost:9200/my-index-000004/_delete_by_query?conflicts=proceed&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'
此时删除了索引my-index-000001内的所有数据，但是这个索引并没有删除，还保留。
conflicts如果被查询删除导致版本冲突，该怎么办： abort或proceed。默认为abort
```

+ 删除多个索引内的全部内容

```shell
curl -X POST "localhost:9200/my-index-000001,my-index-000002/_delete_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'
```

+ 路由删除(暂时不懂)

```shell
curl -X POST "localhost:9200/my-index-000001/_delete_by_query?routing=1&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range" : {
        "age" : {
           "gte" : 10
        }
    }
  }
}
'
```

+ 调整删除总量

```shell
curl -X POST "localhost:9200/my-index-000001/_delete_by_query?scroll_size=5000&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "term": {
      "user.id": "kimchy"
    }
  }
}
'
```

+ 查看删除状态

```shell
curl -X GET "localhost:9200/_refresh?pretty"
curl -X POST "localhost:9200/my-index-000001/_search?size=0&filter_path=hits.total&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "http.response.bytes": {
        "lt": 2000000
      }
    }
  }
}
'
```





###  UPDATE API(更新值)

+ 基本请求格式

```
POST /<index>/_update/<_id>
```

+ 更新字段的数量

```shell
curl -X POST "localhost:9200/my-index-000004/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script" : {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params" : {
      "count" : 4
    }
  }
}
'
```

+ 增加标签

```shell
curl -X POST "localhost:9200/my-index-000004/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.tags.add(params.tag)",
    "lang": "painless",
    "params": {
      "tag": "blue"
    }
  }
}
'
原标签字段会成为一个列表
```

+ 修改字段值(或添加新的字段)

```shell
curl -X POST "localhost:9200/my-index-000004/_update/10?pretty" -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.new_field = \u0027value_of_new_field\u0027"
}
'
```

+ 插入新的字段(跟上述的功能一样)

```shell
curl -X POST "localhost:9200/my-index-000004/_update/10?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": {
    "name": "new_name"
  }
}
'
```

+ 删除新的字段

```shell
curl -X POST "localhost:9200/my-index-000004/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script" : "ctx._source.remove(\u0027new_field\u0027)"
}
'
```

+ 增补

如果文档尚不存在，则`upsert`元素的内容将作为新文档插入。如果文档存在， `script`则执行：

```shell
curl -X POST "localhost:9200/my-index-000004/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.counter += params.count",
    "lang": "painless",
    "params": {
      "count": 4
    }
  },
  "upsert": {
    "counter": 1
  }
}
'
```



###  UPDATE BY QUERY API(通过搜索来更新)

+ 基本格式

```
POST /<target>/_update_by_query
```

+ 查询修改，没查到就是新增

```shell
curl -X POST "localhost:9200/my-index-000001/_update_by_query?refresh&slices=5&pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source[\u0027extra\u0027] = \u0027test\u0027"
  }
}
'
```





###  GET API(获取一个索引值)

**参数**

+ _source=false  将不带资源详情
+ _source_includes 和 source_excludes   包含和过滤

只想表示包含

```
curl -X GET "localhost:9200/my-index-000001/_doc/0?_source=*.id&pretty"

此时_source 中只有user.id
```

仅获取源字段

```
curl -X GET "localhost:9200/my-index-000001/_source/1?pretty"
```

使用过滤参数

```
curl -X GET "localhost:9200/my-index-000001/_source/1/?_source_includes=*.id&_source_excludes=entities&pretty"
表示只取id排除entities
```

###  mget API(多获取api)

+ 通过id获取数据

```shell
curl -X GET "localhost:9200/_mget?pretty" -H 'Content-Type: application/json' -d'
{
  "docs": [
    {
      "_index": "my-index-000001",
      "_id": "1"
    },
    {
      "_index": "my-index-000001",
      "_id": "2"
    }
  ]
}
'
```

+ 同上

```shell
curl -X GET "localhost:9200/my-index-000001/_mget?pretty" -H 'Content-Type: application/json' -d'
{
  "docs": [
    {
      "_type": "_doc",
      "_id": "1"
    },
    {
      "_type": "_doc",
      "_id": "2"
    }
  ]
}
'
```

+ 同上

```shell
curl -X GET "localhost:9200/my-index-000004/_doc/_mget?pretty" -H 'Content-Type: application/json' -d'
{
  "docs": [
    {
      "_id": "1"
    },
    {
      "_id": "2"
    }
  ]
}
'
```

+ 同上

```shell
curl -X GET "localhost:9200/my-index-000001/_mget?pretty" -H 'Content-Type: application/json' -d'
{
  "ids" : ["1", "2"]
}
'
```

+ 根据不同需求查询

```shell
curl -X GET "localhost:9200/_mget?pretty" -H 'Content-Type: application/json' -d'
{
  "docs": [
    {
      "_index": "test",
      "_type": "_doc",
      "_id": "1",
      "_source": false
    },
    {
      "_index": "test",
      "_type": "_doc",
      "_id": "2",
      "_source": [ "field3", "field4" ]
    },
    {
      "_index": "test",
      "_type": "_doc",
      "_id": "3",
      "_source": {
        "include": [ "user" ],
        "exclude": [ "user.location" ]
      }
    }
  ]
}
'
```







