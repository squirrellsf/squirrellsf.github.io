---
layout: post
categories: Java
title: "Macbook利用brew安装MongoDB"
tags: [MongoDB]
date-string: 2018-09-17
---
# Macbook 利用brew 安装MongoDB
MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。
Macbook安装MongoDB的方法有三种：
* 手动解压安装；
* homebrew安装；
* 下载安装包。    
由于UNIX内核的命令行功能，因此采用了homebrew安装MongoDB，具体步骤如下：
1. 首先更新homebrew
```
   brew update
```
2. 安装MongoDB
```
   brew install mongodb
```
3. 默认mongodb 数据文件是放到根目录 data/db 文件夹下,如果没有这个文件,需自行创建。
```
   mkdir -p /data/db
```
   需要注意的是，macbook默认/data/db文件夹是只读的，因此需要手动更改/data/db文件夹的访问权限。这里也有两种方法：①chmod命令；②.图形化界面更改权限

4. 将mongod加入到环境变量，并立即生效
```
    open -e ~/.bash_profile
```
    在环境变量中添加：
```
     export PATH=/usr/local/Cellar/mongodb/4.0.2/bin:${PATH}
```
    数字4.0.2表示MongoDB的版本号。
    生效：
```
    source ~/.bash_profile
```
5. 修改mongodb配置文件,配置文件默认在 /usr/local/etc 下的 mongod.conf。
         dbpath = /data/db表示数据库文件写入的目录地址
6. 启动服务器
```
    mongod
```
7. 启动客户端
```
    mongo
```

