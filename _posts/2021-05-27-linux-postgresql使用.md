---
layout: post
title: "linux笔记: postgresql使用"
subtitle: "pgs简单命令使用"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux

---

## 命令

| 命令描述                 | mysql                    | postgresql                                         |
| ------------------------ | ------------------------ | -------------------------------------------------- |
| client命令               | mysql                    | psql                                               |
| 列出数据库               | show databases;          | \l ; \list ;                                       |
| 选择数据库               | use demo;                | \c demo;                                           |
| 列出当前数据库的所有表格 | show tables;             | select * from pg_tables where schemaname='public'; |
| 列出当前表格中的值       | select * from demotable; | select * from demotable;                           |
| 当前表格结构             | desc demotable;          | \d demotable;                                      |

