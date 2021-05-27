---
layout: post
title: "docker笔记: mtu未对齐导致https访问不通问题处理"
subtitle: "docker中访问https不通问题处理"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux

---

## 问题

现象：容器内访问域名http可以，https不通

解决：

1. 获取docker 的Pid号：``docker top id`` ``docker inspect id``
2. 进入该容器的网络空间：``nsenter -n -t pid``
3. 设置mtu为指定值：``ifconfig 网卡 mtu 数值 up``

