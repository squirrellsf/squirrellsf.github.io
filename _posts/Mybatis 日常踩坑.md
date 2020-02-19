---
layout: post
categories: Java
title: "MyBatis 踩坑记录"
tags: [Java, MyBatis]
date: 2019-09-03
---
# MyBatis 日常踩坑
*注解版MyBatis获取自增主键*
在使用MyBatis注解版，往数据库里边Insert文档的时候，一般都不设置id，而是使用自增主键。这时候自增主键在之后的代码中会有更大的作用，所以需要在插入之后获得这个自增主键。
方法有两种：
1. 使用select last_insert_id()
>    @Insert("insert into table(content,questionid,active,submit,createtime) values(#{content},#{questionid},  
>             1,#{submit},#{createtime})")
>    @SelectKey(statement="select last_insert_id()"
>                    ,before=false,keyProperty="_id",resultType=Integer.class,keyColumn="_id")
>    int insertQuestionItem(QuestionItemInfo questionItemInfo);
   其中，keyProperty是对象中想要的数据名（不一定是id，可以是别的字段，但返回的一定是最后插入的内容），keyColumn是对应的数据库字段名.
   
>    @InsertProvider(type=OperateSqlProvider.class,method="insertQuestion")
> 	@SelectKey(statement="select last_insert_id()",
> 	before=false,keyProperty="bean._id",resultType=Integer.class,keyColumn="_id")
> 	int insertQuestion(@Param("bean")QuestionInfo questionInfo);
这是使用Provider代理处理，使用了注解(@Param)，这时keyProperty发生了变化，变为了 【Param名称.属性名称】
在使用provider之后，要在provider方法中进行响应的设置：
> @Override
> public int insertQuestion(QuestionInfo bean){
>     mapper.insertQuestion(bean);//MyBatis操纵数据库，返回的信息已经封装到了bean内。
>     return bean.getId();
> }
    
2. 使用options注解
   这种方法较为简洁，即：
   1. 写SQL，但不要自己插入主键值 
   2. 配置@Options(useGeneratedKeys=true, keyProperty="对象.属性") 这个的作用是设置是否使用JDBC的getGenereatedKeys()方法获取主键并赋值到keyProperty设置的对象的属性中，说白了就是把自增长的主键值赋值给对象相应的属性 
   3. 在插入后，使用对象.主键属性的getXXId()方法 获取主键值

>    @InsertProvider(type=OperateSqlProvider.class,method="insertQuestion")
> 	  @Options(useGeneratedKeys = true, keyProperty = "bean._id")
> 	  int insertQuestion(@Param("bean")QuestionInfo questionInfo);
	


