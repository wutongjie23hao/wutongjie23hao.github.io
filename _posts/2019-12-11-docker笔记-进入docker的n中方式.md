---
layout: post
title: "docker笔记: 交互式方式进入docker的n种方式"
subtitle: "docker exec/ nsenter 以交互式方式进入docker"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---
## 方法
* ``nsenter --target <docker-pid> --mount --uts --ipc --net --pid`` 
```
# docker inspect -f '{{ .State.Pid }}' 502d140f3c62 
480700
# nsenter --target 480700 --mount --uts --ipc --net --pid
[root@helo-hjzfj /]#
```
* ``docker exec -it <docker-id> bash``
```
# docker exec -it 502d140f3c62 bash
[root@hello-hjzfj /]# 
```

## 备注
一般在docker exec出现问题，比如docker卡死的时候，磁盘故障的时候，等等，需要使用nsenter。