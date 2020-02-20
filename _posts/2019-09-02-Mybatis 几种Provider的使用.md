---
layout: post
categories: Java
title: "MyBatis几种Provider的使用"
tags: [MyBatis]
date-string: 2019-09-02
---
# Mybatis 几种Provider的使用

之前在Mybatis踩坑的文章中提到了使用provider的方法， 那里边使用的是通过Map<String,Object>传递bean，在provider中再取出bean,接着设置value的方法。这种方法不能很好地防止sql注入的问题，比如：
```
 if(paper.getAuthor()!=null){
     sql.VALUES("authors","'"+paper.getAuthor()+"'");
 }
 ```
这条语句如果不在paper.getAuthor前后加单引号的话，就可能出现sql注入，导致出现sql语句的语法错误。
其实在使用provider时可以不通过Map<String,Object>传递参数，直接传bean，既可以不用在provider中再去取出bean，也能避免注入的问题。这样在provider中，可以直接用#{}取出bean对应的属性。

参考链接：https://www.cnblogs.com/JoeyWong/p/9457118.html

