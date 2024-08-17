

默认情况下，ES会对文档中每个字段的全部数据进行索引，每个索引字段都有一个专门的、优化过的数据结构，例如text字段存储在倒排索引中，数字和GEO字段存储在BKD树中。

# Mapping

ES可以是无schema的，前提是使用dynamic mapping

mapping非常重要，通常需要精心指定。

## 多字段fields

当同一个字段被用于多种目的时，例如一个字符串类型的字段，可能用于全文搜索，也可能被用于排序或者聚合。这个功能通过fields参数支持，即多字段功能multi-fields：

例如在下面的city字段，它本身是text类型（用于全文搜索），可以通过fields指定一个city.raw字段，它是keyword类型（用于排序或聚合），即keyword版本的city：

~~~
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "city": {
        "type": "text",
        "fields": {
          "raw": { 
            "type":  "keyword"
          }
        }
      }
    }
  }
}
~~~

下面是一个使用city进行搜索，使用city.raw进行排序和聚合的例子：

~~~
GET my-index-000001/_search
{
  "query": {
    "match": {
      "city": "york" 
    }
  },
  "sort": {
    "city.raw": "asc" 
  },
  "aggs": {
    "Cities": {
      "terms": {
        "field": "city.raw" 
      }
    }
  }
}
~~~

fields的另一个作用是给相同的字段指定不同的分词器，例如下面的text字段默认使用标准分词器，而text.english字段则单独指定了英语的分词器：

~~~
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "text": { 
        "type": "text",
        "fields": {
          "english": { 
            "type":     "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}
~~~

查询时，可以同时查询text字段，以及text.english字段：

~~~
GET my-index-000001/_search
{
  "query": {
    "multi_match": {
      "query": "quick brown foxes",
      "fields": [ 
        "text",
        "text.english"
      ],
      "type": "most_fields" 
    }
  }
}

~~~

## text类型

text是用于全文搜索的，这种类型的字段在索引前会将字符串类型，通过分词器，转换为多个词的列表

可以给text设置fielddata=true，用于支持排序、聚合，它会将倒排索引转换为一种适合排序或聚合的结构，但可能会占用大量内存

也可以通过多字段功能实现排序和聚合：

~~~
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "my_field": { 
        "type": "text",
        "fields": {
          "keyword": { 
            "type": "keyword"
          }
        }
      }
    }
  }
}
~~~

## keyword类型

keyword用于支持排序、聚合、精确查询（Term-level queries）：

~~~
PUT my-index-000001
{
  "mappings": {
    "properties": {
      "tags": {
        "type":  "keyword"
      }
    }
  }
}
~~~

一个数字类型的字段也可以设置为keyword类型，前提是不需要range查询

# Query

ES查询语言DSL (Domain Specific Language) 是一种基于json的查询语言，它由两类语法组成：

* 叶子查询从句：可以单独使用的查询，它们用于在字段中查找特定值，如match、term、range
* 复合查询从句：它包含叶子查询从句，可以包含多个查询条件，或者更改查询的行为，如bool、dis_max、constant_score

## Context

查询中通常由两种Context：

* Query Context：使用它的时候，会伴随着相关性算分的计算，它会查询复合要求的文档，并返回_score
* Filter Context：使用它的时候，不会计算相关性算分，它仅仅只是过滤符合要求的文档。使用Filter Context时ES还可以借助缓存来优化查询速度

下面是一个context生效的例子：

~~~
GET /_search
{
  "query": { 
    "bool": { 
      "must": [
        { "match": { "title":   "Search"        }},
        { "match": { "content": "Elasticsearch" }}
      ],
      "filter": [ 
        { "term":  { "status": "published" }},
        { "range": { "publish_date": { "gte": "2015-01-01" }}}
      ]
    }
  }
}
~~~

查询语句中的第一个query代表了它是一个query context，所以在query context下的两个match子句都计算相关性算分；而filter代表它是一个filter context，所以在它下面的两个子句term和range都不会计算相关性算分

## 查询总记录数

在查询时默认是不查总数的，因为查询记录总数的开销比较大，可以通过设置track_total_hits为true来查询精确的总数：

~~~
GET my-index-000001/_search
{
  "track_total_hits": true,
  "query": {
    "match" : {
      "user.id" : "elkbee"
    }
  }
}
~~~

上面的请求会返回：

~~~
{
  "_shards": ...
  "timed_out": false,
  "took": 100,
  "hits": {
    "max_score": 1.0,
    "total" : {
      "value": 2048,    
      "relation": "eq"  
    },
    "hits": ...
  }
}
~~~

其中total.value代表总数，total.relation为eq代表当前的总数是精确的，当track_total_hits为true时，total.relation总是eq，它也可能是gte，代表是总数的下边界

track_total_hits也可以设置为一个整数，代表最多匹配多少个文档

## 全文搜索

Match query是全文搜索的标准形式，这种查询的入参，在进行匹配前会先分词：

~~~
GET /_search
{
  "query": {
    "match": {
      "message": {
        "query": "this is a test"
      }
    }
  }
}
~~~



## 精确查询

Term-level queries用来精确匹配

其中最常用的一种是Term Query，注意不要在text字段上使用Term Query，因为ES默认会将text字段转换为一个列表，这会使精确匹配变得困难，查询text字段通常使用Match Query

下面是一个例子，针对user.id字段，找出值包含kimchy的文档，boost是用来控制返回值中的相关性评分的：

~~~
GET /_search
{
  "query": {
    "term": {
      "user.id": {
        "value": "kimchy",
        "boost": 1.0
      }
    }
  }
}
~~~

例如有个文档的full_text（text类型）字段值是Quick Brown Foxes!：

~~~
PUT my-index-000001/_doc/1
{
  "full_text":   "Quick Brown Foxes!"
}
~~~

ES会将它转换为一个数组：[quick, brown, fox]

使用下列Term Query会查找不到数据，因为在数组中找不到精确相等的情况：

~~~
GET my-index-000001/_search?pretty
{
  "query": {
    "term": {
      "full_text": "Quick Brown Foxes!"
    }
  }
}
~~~

而使用Match Query就会有值，因为它会将查询入参分词为三个词：quick、brown 、fox，会找到包含这三个词中至少任意一个的文档：

~~~
GET my-index-000001/_search?pretty
{
  "query": {
    "match": {
      "full_text": "Quick Brown Foxes!"
    }
  }
}
~~~

Terms Query可以同时匹配多个值，例如下面的例子，找出user.id包含kimchy或elkbee的文档：

~~~
GET /_search
{
  "query": {
    "terms": {
      "user.id": [ "kimchy", "elkbee" ],
      "boost": 1.0
    }
  }
}
~~~

Terms Query可以查找现有文档的字段值，然后用这些字段值再去进行查询（Terms lookup），如下面的例子：

~~~
GET my-index-000001/_search?pretty
{
  "query": {
    "terms": {
        "color" : {
            "index" : "my-index-000001",
            "id" : "2",
            "path" : "color"
        }
    }
  }
}
~~~

先找到index为my-index-000001，id为2的，列为color的值，然后用这些值为入参进行查询

## 复合查询

复合查询中最常用的是Bool Query，它包含几个子句：must、filter、should和must_not

minimum_should_match参数是用来指定should中，到底有多少是至少要满足的。如果bool查询中不包含must或者filter，那这个值是1，否则就是0







ES REST API支持结构化查询和全文搜索查询两种（或者是两者结合）：

* 结构化查询类似于SQL，可以指定查询字段、按xx排序等
* 全文搜索查询用于搜索，一般按照相关性排序优先返回

通过文档ID查找：

~~~
GET /customer/_doc/1
~~~

查询索引bank的全部文档，按account_number字段排序：

~~~
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ]
}
~~~

分页查询：

~~~
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": [
    { "account_number": "asc" }
  ],
  "from": 10,
  "size": 10
}
~~~

match query，查询address包含mill或者lane的：

~~~
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
~~~

match_phrase query，查询address包含 mill lane的：

~~~
GET /bank/_search
{
  "query": { "match_phrase": { "address": "mill lane" } }
}
~~~

bool query可以整合多个查询条件：

~~~
GET /bank/_search
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
~~~

其中must和should是贡献算分的，must_not不贡献算分，must_not其实可以被视为是一种filter，filter可以指定包含或者不包含结果：

~~~
GET /bank/_search
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
~~~

Terms Aggregation：按state字段分组，默认返回分组后文档数量最多的10条记录：

~~~
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
~~~

在分组的基础上，计算字段balance的平均值：

~~~
GET /bank/_search
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
~~~

指定按结果中的字段排序，而不是按默认的文档数排序：

~~~
GET /bank/_search
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
~~~

创建索引时指定mapping：

~~~
PUT /my-index-000001
{
  "mappings": {
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  }, 
      "name":   { "type": "text"  }     
    }
  }
}
~~~

添加一个keyword类型的字段，index为false代表这个字段存储但是不会索引，也不支持搜索：

~~~
PUT /my-index-000001/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false
    }
  }
}
~~~

查看索引的mapping：

~~~
GET /my-index-000001/_mapping
~~~

















