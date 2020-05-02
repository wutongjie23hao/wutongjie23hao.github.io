---
layout: post
title: "linux笔记: mysql启动报错处理"
subtitle: "报错：The server quit without updating PID file/Cannot open tablespace"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux

---

# 记一次mysql报错的处理

## 1. 现象

1. 终端报错：

   ```
   > /etc/init.d/mysqld start
   Starting MySQL.. ERROR! 
   The server quit without updating PID file 
   ```

1. 日志：

   ```
   此时去查看error日志，错误如下：
   Cannot open tablespace xxxx
   ```

## 2. 解决

修改mysqld的配置文件，其中[mysqld]部分添加：

```
innodb_force_recovery = 1
```

然后启动成功，但是，此时，数据库为只读权限，需要设置成读写并刷新权限：

```
# mysql
mysql> set global read_only=0;
mysql> flush privileges;
```

## 3. 一些mysql命令

1. mysql授权

   ```
   mysql> grant all on * to 'username'@'%' identified by 'password';
   mysql> set global read_only=0;
   mysql> flush privileges;
   ```
