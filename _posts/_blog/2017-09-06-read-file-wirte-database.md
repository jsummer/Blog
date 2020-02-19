---
layout: post
title:  "读取文件数据小结"
date:   2017-09-06 18:00:00 
categories: blog
tags: ['node','mysql','数据库']
published: true
comments: true
script: [post.js]
---

* Do not remove this line (it will not be displayed)
{: toc}

## 处理过程

入职第二天，郭导上午给我发了个任务：

> 我建了一张表    test_lihao_20170906
    需求描述：
    将附件数据按照竖线分割导入到mysql库中，建表语句按照下面的字段顺序。

看了下需求，先大概有个思路。
读文件 =》提取数据 =》写入数据库
好，就这样做。

先看看文件结构，一行就是一条数据，每个字段数据以 `|` 分割，结构比较简单。

> 提取数据难度 🌟
使用js String.prototype.split 
正则都不需要，so easy

下来想想用什么来读取数据，java? node? shell?
java算了，太陌生了。
node可以，使用fs模块就可以
shell，刚才正在看这个（打算把bat文件转成sh文件），可以试试，好，那就试试。

于是，我就开始用shell，学习了下怎么连接mysql，怎么连接mysql，怎么连接mysql，我就卡壳了

```
#!/bin/bash
##############################
# @file create_db_mysql.sh
# @brief create database and tables in mysql
# @author Mingxing LAI
# @version 0.1
# @date 2013-01-20
##############################
USER="root"
DATABASE="students"
TABLE="students"
######################
#crate database
mysql -u $USER << EOF 2>/dev/null
CREATE DATABASE $DATABASE
EOF
[ $? -eq 0 ] && echo "created DB" || echo DB already exists
######################
#create table
mysql -u $USER $DATABASE << EOF 2>/dev/null
CREATE TABLE $TABLE(
id int,
name varchar(100),
mark int,
dept varchar(4)
);
EOF
[ $? -eq 0 ] && echo "Created table students" || echo "Table students already exist" 
######################
#delete data
mysql -u $USER $DATABASE << EOF 2>/dev/null
DELETE FROM $TABLE;
EOF

```

`<< EOF 2 >/dev/null`, 这是什么鬼，除去sql部分，其他根本看不懂，好吧，只能用node了，要尽快完成任务。

与其同时，我还在下载mysql，因为本地没有mysql，看了看进度，3%，oh shit，还好手机流量还有6个G。

用node来写，我首先想到的是 `fs` 模块，通过识别换行符，来取出每行数据。
在查资料的过程中发现有个 `readline` 模块，直接可以将每行数据取出来，我的原则是哪个快就用哪个。

所以读取文件，提取数据也这么愉快的结束了。

> 读取文件，提取数据 🌟

最后一步，将数据写入数据库，还好前段时间写了写sql。
连接数据库自然不在话下，通过 `npm install mysql`，调用 `mysql`来和数据库连接。

insert 语句也写好了，19个字段，来运行一下，报错，没有这个字段，查下表吧，发现确实没有这个字段，表里只有两个字段，一个自增ID，一个test，先改表吧，加字段，数据类型全部为 `varchar(255)` (这里有坑)，可以了吧，再来。

嗯，这次愉快的添加进去了，查下表的数据总条数，1630，卧槽，不对呀，一共1690行，没有全部加进去，看日志，有报错，数据长度过长，原来设置的长度不够，那长度设置个1000吧，等等，脑子瞬间冒出两个问题，一个是我不知道是哪个字段的长度过长，还有1000够吗？

啪啪啪，处理完毕，查找出了超过255的字段名称以及最大长度，最长的将近1000，小伙子，第六感不错。

修改对应字段长度，再次运行，查询数据条数，1690，可以，打完收工。

> 写入数据库 🌟🌟

记录下用到的command吧

|名称|command|
|--|--|
|远程连接mysql服务器|mysql -u username -p -h domain -P port|
|本地连接mysql服务器|mysql -u username -p|
|使用数据库|use databaseName;|
|显示数据库列表|show databases;|
|显示库中的表列表|show tables;|
|显示表信息|desc tableName;|
|添加表|create table tableName(fieldName dataType(size));|
|删除表| drop table tableName;|
|向表中插入记录|insert into tableName (fieldName1, fieldName2, ...) values(value1, value2, ...);|
|清空表记录|delete from tableName;|
|显示表记录|select * from tableName;|
|显示表大小|select count(*) from tableName;|
|显示表中的一条信息|select * from tableName limit 1;|
|修改表字段信息(可修改名称)|alter table tableName change fieldOldName fieldNewName dataType(size) null;|
|修改表字段信息|alter table tableName modify fieldName dataType(size) null;|
|添加表字段信息|alter table tableName add fieldName dataType(size) null;|
|修改自增ID初始值|alter table tableName AUTO_INCREMENT=0;|

## 后记

1.源数据 是没有字段缺少情况的，这在向数据库写入的时候提供了极大的便利。
但是如果字段缺少的话，又该怎么处理？

2.读取文件使用的是 `readline` 模块，如果使用 `fs` ，应该怎么写？

3.尝试用其他语言实现相同功能，比如：java,shell



```
const readline = require('readline')
const mysql = require('mysql')
const config = {
  host: '',
  user: '',
  password: '',
  port: '',
  database: ''
}

const fieldArr = ['remote_addr', 'time_local', 'uri', 'args', 'status', 'body_bytes_sent', 'http_referer', 'http_user_agent', '字段1', '字段2', '字段3', 'remote_user', 'request', 'request_time', 'content_length', 'request_uri', 'http_x_forwarded_for', 'request_body', 'upstream_response_time']

// let addSql = `INSERT INTO test_lihao_20170906(remote_addr, time_local, uri, args, status, body_bytes_sent, http_referer, http_user_agent, 字段1, 字段2, 字段3, remote_user, request, request_time, content_length, request_uri, http_x_forwarded_for, request_body, upstream_response_time) VALUES(?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`

let addSql = `INSERT INTO test_lihao_20170906(${fieldArr.join(', ')}) VALUES(?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)`

let connection = mysql.createConnection(config)

// connection.connect()  

let rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  terminal: false
})

let i = 0
let longObj = {}
let longArr = []
let lengthArr = []

rl.on('line', (line) => {
  let arr = line.split('|')
  let len = arr.length
  arr.forEach((item, idx) => {
    if (item.length > 255) {
      if (longObj[idx]) {
        longObj[idx]['max'] = Math.max(longObj[idx]['max'], item.length)
      } else {
        longObj[idx] = {}
        longObj[idx]['max'] = item.length
      }
      if (longArr.indexOf(idx) <= -1) {
        longArr.push(idx)
      }
    }
  })
  if (len < 19) {
    console.log('缺少字段')
    return
  } else {
    i++
    let addSqlParams = arr
    insertData(addSqlParams)
    console.log(i)
  }
  
})

rl.on('close', () => {
  console.log('bye bye!')
  // console.log(longArr)
  for (let key in longObj) {
    // console.log(fieldArr[key], longObj[key].max)
  }
  // 关闭数据库连接
  connection.end()
})

// 向数据库添加数据
function insertData (addSqlParams) {
  connection.query(addSql, addSqlParams, function (err, result) {
    if (err) {
      console.log('[INSERT ERROR] - ', err.message)
      return
    }
    console.log('-------------INSERT-----------')
    console.log('INSERT ID:', result)
    console.log('------------------------------\n\n')
  })
}
```
{% endpost #9D9D9D %}