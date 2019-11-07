---
layout: post
title: "docker笔记: 在容器中使用docker命令"
subtitle: "docker in docker 实践"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---
# 在容器中使用docker命令
```bash
$ docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker -ti centos:7
[root@5fcd5007c70e /]# docker ps
```
> $ docker run -ti -v /var/run/docker.sock:/var/run/docker.sock -v /path/to/static-docker-binary:/usr/bin/docker busybox sh
# 在centos7环境下使用docker in docker
```bash
[root@hello ~]#  docker run --rm -v /var/run/docker.sock:/var/run/docker.sock -v /usr/bin/docker-current:/bin/docker -ti centos:7
[root@5fcd5007c70e /]# yum -y install libseccomp
[root@5fcd5007c70e /]# docker ps
```

> 在宿主机是centos7的物理机上启动docker in docker，出现了两个问题：
1. /usr/bin/docker 不是可执行文件，是一个脚本
2. 报错``libseccomp.so.2: cannot open shared object file: No such file or directory``，缺少这个seccomp静态文件，需要在容器内安装``libseccomp``

# reference
* [https://blog.csdn.net/ygqygq2/article/details/77840501](https://blog.csdn.net/ygqygq2/article/details/77840501)
* [https://zhuanlan.zhihu.com/p/27208085](https://zhuanlan.zhihu.com/p/27208085)