---
layout: post
title: "linux笔记: mysql死锁处理"
subtitle: "mysql死锁处理"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux
---

## mysql死锁处理

### 1. 查看线程

```
 show processlist;
```

### 2. 杀死异常线程

```
kill id;
```

### 3. 查看是否有死锁

```
show OPEN TABLES where In_use > 0;
```

### 4. 关于事务

1. 查看正在占有锁的事务: ``SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS; ``
2. 查看等着锁的事务：``SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS; ``

### 5. 查看锁阻塞线程信息

#### 5.1 使用innodb_lock_monitor来获取阻塞锁线程

```mysql
MySQL [test]> CREATE TABLE innodb_lock_monitor (a INT) ENGINE=INNODB;  ## 随便在一个数据库中创建这个表，就会打开lock monitor  
Query OK, 0 rows affected, 1 warning (0.07 sec)  
  
MySQL [test]> show warnings\G  
*************************** 1. row ***************************  
  Level: Warning  
   Code: 131  
Message: Using the table name innodb_lock_monitor to enable diagnostic output is deprecated and may be removed in future releases. Use INFORMATION_SCHEMA or PERFORMANCE_SCHEMA tables or SET GLOBAL innodb_status_output=ON.  
1 row in set (0.00 sec)  
MySQL [test]> show engine innodb status; # 使用show engine innodb status查看
```

####  5.2 使用mysqladmin debug查看

```mysql
mysqladmin -S /tmp/mysql3306.sock debug
```

然后在error日志中，会看到：

```
Thread database.table_name          Locked/Waiting        Lock_type  
  
  
3       test.t3                     Locked - read         Low priority read lock  
7       test.emp                    Locked - write        High priority write lock 
```

这种方法中，能找到线程ID=3和7是阻塞者，但还是不太准确，判断不出来线程7也是被线程ID=3阻塞的。

### 6. 结论

在分析innodb中锁阻塞时，几种方法的对比情况：

（1）使用show processlist查看不靠谱；
（2）直接使用show engine innodb status查看，无法判断到问题的根因；
（3）使用mysqladmin debug查看，能看到所有产生锁的线程，但无法判断哪个才是根因；
（4）开启innodb_lock_monitor后，再使用show engine innodb status查看，能够找到锁阻塞的根因。

> 原文： https://www.cnblogs.com/duanxz/p/4394641.html

