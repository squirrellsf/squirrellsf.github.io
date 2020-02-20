---
layout: post
categories: Java
title: "ElasticSearch开始(4)"
tags: [ElasticSearch]
date-string: 2018-10-30
---
# ElasticSearch开始（4）--请求体查询

### 查询表达式

### 最重要的查询
#### match_all
match_all 查询简单的 匹配所有文档。
```
{ "match_all": {}}
```

### bool查询
bool查询由一个或者多个子句组成，每个子句都有特定的类型。
#### must
返回的文档必须满足must子句的条件，并且参与计算分值
#### filter
返回的文档必须满足filter子句的条件。但是不会像Must一样，参与计算分值
#### should
返回的文档可能满足should子句的条件。在一个Bool查询中，如果没有must或者filter，有一个或者多个should子句，那么只要满足一个就可以返回。minimum_should_match参数定义了至少满足几个子句。
#### must_not
返回的文档必须不满足must_not定义的条件。

