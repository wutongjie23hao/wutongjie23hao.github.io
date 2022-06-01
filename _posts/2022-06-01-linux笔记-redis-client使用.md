---
layout: post
title: "linux笔记: 清空redis数据"
subtitle: "redis cli 清空redis"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux

---
# cmd

```bash
# redis-cli -h <host> -p <port> -a <password>
redis 127.0.0.1:6379> select 1                 # 切换至1号数据库
redis 127.0.0.1:6379> dbsize                   # 查看数据库数据量
redis 127.0.0.1:6379> flushdb                  # 清空本库的keys
OK
redis 127.0.0.1:6379> keys *                   # keys 列表
```

