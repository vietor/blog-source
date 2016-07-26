---
title: "在ElasticSearch中使用类似GroupBy统计功能"
date: 2016-04-07 14:20:00 +0800
categories: [bigdata]
tags: [elasticsearch]
---

**ElasticSarch**提供了一些基本的聚合统计功能。

**group by**类似的功能，可以使用**aggregations**实现。下面的样例描述了一种通过API调用次数作为**“某类用户行为DAU”**的方法。

* 数据样本
``` json
{
    "@timestamp": "2016-03-14T18:41:36+08:00",
    "@version": "1",
    "@fields": {
        "uri": "/3/query/first",
        "device_id": "108ccf09983e5db88a2d5085e0805195f891e449",
        "app_id": "1000"
    }
}
```

* 聚合需求

将uri为“/3/query/first”的请求，统计出不同app_id中独立deivce_id的访问次数

* 请求实例

``` json
{
    "size": 0,
    "query": {
        "bool": {
            "must": [
                {
                    "term": {
                        "@fields.uri": "/3/query/first"
                    }
                }
            ]
        }
    },
    "aggregations": {
        "g1": {
            "terms": {
                "field": "@fields.app_id"
            },
            "aggregations": {
                "g2": {
                    "cardinality": {
                        "field": "@fields.device_id"
                    }
                }
            }
        }
    }
}
```

* 返回实例

``` json
{
    "took": 696,
    "timed_out": false,
    "_shards": {
        "total": 6,
        "successful": 6,
        "failed": 0
    },
    "hits": {
        "total": 26,
        "max_score": 0,
        "hits": []
    },
    "aggregations": {
        "g1": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": "10000",
                    "doc_count": 10,
                    "g2": {
                        "value": 2
                    }
                },
                {
                    "key": "10002",
                    "doc_count": 9,
                    "g2": {
                        "value": 1
                    }
                }
            ]
        }
    }
}
```

其中“aggregations.g1.buckets”数组中就是获取的数据。key为app_id的值，doc_count为总数，g2.value为独立的device_id的数。


* 功能分析

*ElasticSearch*的aggregations功能能够嵌套使用，能够实现多“字段”的*group by*统计查询，虽然功能行较弱，但的确能够完成一些场景的**统计需求**。
