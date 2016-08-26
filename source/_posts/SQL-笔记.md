---
title: SQL 笔记
date: 2016-08-26 13:48:37
tags: [SQL]
category: [学习笔记]
---
这篇文章主要内容是关于 SQL 语句的, 因为现在做的项目主要就是从网页端操作数据库的东西, 所以要补一补了.
<!--more-->
## limit
```SQL
SELECT * FROM db_pub LIMIT start_row, page_size;
```
以上的 SQL 语句用到了 limit, 它的作用就是从结果集中以 [start_row, page_size] 再筛一遍, 通常配合 `ORDER BY` 使用. 在网页端主要是帮助分页获取数据.