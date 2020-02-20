---
layout: post
categories: Java
title: "ElasticSearch开始(2)"
tags: [ElasticSearch]
date-string: 2018-10-17
---
# ElasticSearch开始（2）--数据输入和输出

 2018.10.17
-------
### 聚合（aggregations)
聚合类似于GROUP BY，但更加强大.
```    
想知道叫 Smith 的雇员中最受欢迎的兴趣爱好：
 GET /megacorp/employee/_search
 {
   "query":{
     "match": {
       "last_name":"smith"
     }
   },
   "aggs": {
     "all_interests": {
       "terms": {
         "field": "interests.keyword"
       }
     }
   }
 }
```
### ES中的文档元数据
#### 1）包含了三个必须的元数据元素：    
①index          
索引，文档在哪儿存放。一个 索引 应该是因共同的特性被分组到一起的文档集合，这个名字必须小写，不能以下划线开头，不能包含逗号。         
②type  
文档表示的对象类别。
③id     
文档唯一标识。ID 是一个字符串， 当它和 index 以及 type 组合就可以唯一确定 Elasticsearch 中的一个文档。可以提供自定义的id值，或者让 index API 自动生成。
  
#### 2）数据输入和输出
##### 为文档建立索引
<HTTP谓词：PUT>    
###### 使用自定义ID【URL中包含了index,type,id】
```
 PUT {index}/{type}/{id}
 {
     "filed":"value",
     ……
 }
 ```
如果index,type,id的组合存在，则会将旧文档覆盖,如果想确保创建新文档，避免覆盖现有文档，则使用_create端点：
```
PUT {index}/{type}/{id}/_create
```

###### 自动生成ID【URL中只包含index,type】
```
 PUT {index}/{type}
 {
     "field":"value"
     ……
 }
 ```
可以确保文档能够被创建

##### 取回一个文档
<HTTP谓词：GET>
###### 仍然使用index,type,id
```
GET {index}/{type}/{id}  
```   
curl命令
```
curl -i -XGET http://localhost:9200/{index}/{type}/{id}   -i：显示响应头
```

##### 返回文档的一部分【使用_source参数】
```
GET {index}/{type}/{id}?_source=filed1,filed2……
```
只返回_source字段，不返回其他元数据【使用_source端点】
```
GET {index}/{type}/{id}/_source
```

##### 检查文档是否存在
```
<HTTP谓词：HEAD>
curl -i -XHEAD http://localhost:9200/{index}/{type}/{id}
```

##### 删除文档
```
<HTTP谓词：DELETE>
DELETE {index}/{type}/{id} 
```

##### 处理冲突
###### 乐观并发控制    
  可以利用_version号来确保应用中相互冲突的变更不会导致数据丢失。通过指定想要修改文档的_version号来达到这个目的。如果该版本不是当前版本号，请求将会失败。
```
 PUT /{index}/{type}/{id}?version=1 
 {
   "title": "My first blog entry",
   "text":  "Starting to get the hang of this..."
 }
```
外部版本号    
<参数：versiontype = external>     
   外部版本号的处理方式和之前的内部版本号的处理方式有些不同，ElasticSearch 不是检查当前_version和请求中指定的版本号是否相同， 而是检查当前_version是否小于指定的版本号。如果是，则请求成功，外部的版本号作为文档的新_version 进行存储。
```
 PUT /website/blog/2?version=5&versiontype=external
 {
   "title": "My first external blog entry",
   "text":  "Starting to get the hang of this..."
 }
```
##### 文档的部分更新
<HTTP谓词：POST>.  
<API:_update>. 
   update 请求最简单的一种形式是接收文档的一部分作为 doc 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。  
###### 增加字段
```
 POST /website/blog/1/_update
 {
    "doc" : {
       "tags" : [ "testing" ],
       "views": 0
    }
 }
```
###### 使用脚本部分更新文档
   脚本可以在 update API中用来改变_source 的字段内容，它在更新脚本中称为ctx._source 。例如，我们可以使用脚本来增加博客文章中 views 的数量： 
```
 POST /website/blog/1/_update
 {
    "script" : "ctx._source.views+=1"
 }
 ```
   也可以通过使用脚本给 tags 数组添加一个新的标签:
```
 POST /website/blog/1/_update
 {
    "script" : "ctx._source.tags.add(params.new_tag)",
    "params" : {
       "new_tag" : "search"
    }
 }
```
###### 更新的文档可能尚不存在
<参数：upsert>. 
  可以使用 upsert 参数，指定如果文档不存在就应该先创建它：
```
 POST /website/pageviews/1/_update
 {
    "script" : "ctx._source.views+=1",
    "upsert": {
        "views": 1
    }
 }
```
##### 取回多个文档
<API:_mget>. 
<参数：docs>. 
   mget API 要求有一个 docs 数组作为参数，每个 元素包含需要检索文档的元数据， 包括index、type 和id 。如果想检索一个或者多个特定的字段，那么你可以通过_source 参数来指定这些字段的名字：
```
 GET /_mget
 {
    "docs" : [
       {
          "index" : "website",
          "type" :  "blog",
          "id" :    2
       },
       {
          "index" : "website",
          "type" :  "pageviews",
          "id" :    1,
          "_source": "views"
       }
    ]
 }
```
##### 代价较小的批量操作
<API:bulk>. 
   bulk API 允许在单个步骤中进行多次 create 、index 、 update 或 delete 请求。 如果你需要索引一个数据流比如日志事件，它可以排队和索引数百或数千批次。
格式：
 { action: { metadata }} <!--指定哪一文档做什么操作  action:"create","index","update","delete"-->
 { request body        }
 { action: { metadata }}
 { request body        }
 ...
metadata 应该指定被索引、创建、更新或者删除的文档的index、type 和id。
```
{"delete":{"index":"","type":"","d":""}}
```
request body行由文档的_source本身组成--文档包含的字段和值。它是 index 和 create 操作所必需的，你必须提供文档以索引。
它也是 update 操作所必需的，并且应该包含你传递给 update API 的相同请求体： doc 、 upsert 、 script 等等。 删除操作不需要 request body 行。
```
 { "create":  { "index": "website", "type": "blog", "id": "123" }}
 { "title":    "My first blog post" }
 ```
如果不指定id,将自动生成一个ID：
```
 { "index": { "index": "website", "type": "blog" }}
 { "title":    "My second blog post" }
 ```
一个完整的bulk请求：
```
POST /_bulk
 { "delete": { "index": "website", "type": "blog", "id": "123" }} ①
 { "create": { "index": "website", "type": "blog", "id": "123" }}
 { "title":    "My first blog post" }
 { "index":  { "index": "website", "type": "blog" }}//其实也是创建一个文档_
 { "title":    "My second blog post" }
 { "update": { "index": "website", "type": "blog", "id": "123", "_retry_on_conflict" : 3} }
 { "doc" : {"title" : "My updated blog post"} } ②
 
//①：delete不能有请求体，其后跟着另一个操作
//②：最后必须有换行符
```



