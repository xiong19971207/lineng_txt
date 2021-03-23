###  Elasticsearch随笔

+ python如何使用elasticsearch

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































