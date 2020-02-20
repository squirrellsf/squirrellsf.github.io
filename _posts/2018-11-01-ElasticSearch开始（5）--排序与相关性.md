---
layout: post
categories: Java
title: "ElasticSearch开始(5)"
tags: [ElasticSearch]
date-string: 2018-11-01
---
# ElasticSearch开始（5）--排序与相关性
**排序**
为了按照相关性来排序，需要将相关性表示为一个数值。在 Elasticsearch 中， 相关性得分 由一个浮点数进行表示，并在搜索结果中通过 _score 参数返回， 默认排序是 _score 降序。

使用filter参数：只希望获取匹配 user_id: 1 的文档，并没有试图确定这些文档的相关性。 实际上文档将按照随机顺序返回，并且每个文档都会评为零分。
*按照字段的值排序*
通过时间来对 tweets 进行排序是有意义的，最新的 tweets 排在最前：
```
 GET /_search
 {
     "query" : {
         "bool" : {
             "filter" : { "term" : { "user_id" : 1 }}
         }
     },
     "sort": { "date": { "order": "desc" }}
 }
 ```
sort和aggs很类似，都是先执行query操作，再获得集合之后，在集合上进行排序操作，所以sort和query是并列的。

*多级排序*
结合使用 date 和 _score 进行查询，并且匹配的结果首先按照日期排序，然后按照相关性排序：
```
 GET /_search
 {
     "query" : {
         "bool" : {
             "must":   { "match": { "tweet": "manage text search" }},
             "filter" : { "term" : { "user_id" : 2 }}
         }
     },
     "sort": [
         { "date":   { "order": "desc" }},
         { "_score": { "order": "desc" }}
     ]
 }
 ```
！！排序条件的顺序是很重要的。结果首先按第一个条件排序，仅当结果集的第一个 sort 值完全相同时才会按照第二个条件进行排序，以此类推。

*多值字段的排序*
mode 选项：min,max,sum,avg,median
```
 POST /_search
 {
    "query" : {
       "term" : { "product" : "chocolate" }
    },
    "sort" : [
       {"price" : {"order" : "asc", "mode" : "avg"}}
    ]
 }
 ```
 
*嵌套对象排序*
nested选项：path,filter,nested
必须指定path字段，否则es不知道在哪个嵌套层上进行排序（进行nested查询时也一样）
```
 POST /_search
 {
    "query" : {
       "term" : { "product" : "chocolate" }
    },
    "sort" : [
        {
           "offer.price" : {
              "mode" :  "avg",
              "order" : "asc",
              "nested": {
                 "path": "offer",
                 "filter": {
                    "term" : { "offer.color" : "blue" }
                 }
              }
           }
        }
     ]
 }
 ```
 
*丢失值*
如果排序字段中没有值，missing参数可以用来设置这些文档的排序方式，可以是_last,_first或用来排序的自定义值。默认为_last。
```
 GET /_search
 {
     "sort" : [
         { "price" : {"missing" : "_last"} }
     ],
     "query" : {
         "term" : { "product" : "chocolate" }
     }
 }
 ```
 
*忽略无映射字段*
unmapped_type选项：如果一个字段在这个Index中并不存在，则使用unmapped_type选项忽略该字段，并将其视为一个指定类型的映射，所有文件在这个字段上的值都为空
```
 GET /_search
 {
     "sort" : [
         { "price" : {"unmapped_type" : "long"} }
     ],
     "query" : {
         "term" : { "product" : "chocolate" }
     }
 }
 ```



