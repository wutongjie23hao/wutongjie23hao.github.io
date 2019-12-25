---
layout: post
title: "docker学习-cgroups资源限制"
subtitle: "cgroups资源限制"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker学习
---
## 树结构
待补充

## cgroup工具
安装：``apt-get install cgroup-bin``

使用：

* 查看所有的 cgroup：lscgroup
* 查看所有支持的子系统：lssubsys -a
* 查看所有子系统挂载的位置： lssubsys –m
* 查看单个子系统（如 memory）挂载位置：lssubsys –m memory

## reference
[https://www.infoq.cn/article/docker-kernel-knowledge-cgroups-resource-isolation/](https://www.infoq.cn/article/docker-kernel-knowledge-cgroups-resource-isolation/)
