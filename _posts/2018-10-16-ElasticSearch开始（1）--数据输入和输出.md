---
layout: post
categories: Java
title: "ElasticSearch开始(1)"
tags: [ElasticSearch]
date-string: 2018-10-16
---
# ElasticSearch开始（1）--数据输入和输出

2018.10.16
-------

1. Query and Filter CONTEXT
Query:决定“此文档与此查询子句的匹配程度如何？”，除了判断文档是否匹配之外，查询子句还计算_score表示文档相对于其他文档的匹配程度。query参数
Filter：决定“此文档是否与此查询子句匹配？”，答案是简单的是与否。filter参数
 ```
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
   ```
 *Tips: Query 和 Filter的实现步骤其实是：①先通过query找到所有符合查询条件的文档；②再从①中检索到的文档集合中进行筛选和过滤（filter），得到最后的结果。
 (其他语法很多都类似，先检索，再筛选（逻辑上和SQL语句很像，先SELECT，再设置其他查询条件））*
 
 2. Match All/None Query
 ```
  GET /_search
  {
     "query":{
         "match_all":{}
     }
  }
  ```
  ```
  GET /_search
  {
     "query":{
         "match_none":{}
         }
  }
 ```
 

