---
layout: post
title:  "数据加载、存储与文件格式"
categories: blog
tags: ['python', '数据分析', 'pandas']
published: true
comments: true
script: [post.js]
---

* Do not remove this line (it will not be displayed)
{: toc}

## 读写文本格式的数据

|函数|说明|
|--|--|
|read_cvs|从文件、URL、文件型对象中加载带分隔符的数据。默认分隔符为逗号|
|read_table|从文件、URL、文件型对象中加载带分隔符的数据。默认分隔符为制表符('\t')|
|read_fwf|读取定宽列格式数据(也就是说没有分隔符)|
|read_clipboard|读取剪贴板中的数据。可以看做是read_table的剪贴板版。在将网页转换为表格时很有用|
|read_excel|从Excel XLS 或 XLSX file 读取表数据|
|read_hdf|读取pandas写的HDF5文件|
|read_html|读取html中的所有表格|
|read_json|读取JSON字符串中的数据|
|read_msgpack|二进制格式编码的pandas数据|
|read_pickle|读取Python pickle 格式中存储的任意对象|
|read_sas|读取存储于SAS系统自定义存储格式的SAS数据集|
|read_sql|(使用SQLAlchemy) 读取SQL查询结果为pandas的DataFrame|
|read_stata|读取Stata文件格式的数据集|
|read_feather|读取Feather二进制文件格式|


* 索引：将一个或多个列当做返回的DataFrame处理，以及是否从文件、用户获取列名。
* 类型推断和数据转换：包括用户自定义值的转换、和自定义的缺失值标记列表等。
* 日期解析：包括组合功能，比如将分散在多个列中的日期时间信息组合成结果中的单个列。
* 迭代：支持对大文件进行逐块迭代
* 不规整数据问题：跳过一些行、页脚、注释或其他一些不重要的东西（比如由成千上万个逗号隔开的数值数据）



{% endpost #9D9D9D %}

