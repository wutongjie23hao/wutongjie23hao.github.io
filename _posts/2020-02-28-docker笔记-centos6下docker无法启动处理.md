---
layout: post
title: "docker笔记: centos6下docker无法启动处理"
subtitle: "记一次centos6下docker无法启动的处理过程"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---
## centos6下无法启动docker

### 1. 启动

```bash
# docker version
Client version: 1.3.0
Client API version: 1.15
Go version (client): go1.3.3
Git commit (client): 39fa2fa/1.3.0
OS/Arch (client): linux/amd64
Server version: 1.3.0
Server API version: 1.15
Go version (server): go1.3.3
Git commit (server): 39fa2fa/1.3.0

# docker -d -H unix:///var/run/docker.sock -H tcp://127.0.0.1:5050 --bridge=none -g /hello/docker --storage-driver=vdisk # 验证docker是否可正常启动,或者直接docker -d启动

```



### 2. 报错

运行``docker -d -H unix:///var/run/docker.sock -H tcp://127.0.0.1:5050 --bridge=none -g /hello/docker --storage-driver=vdisk`` 后发现无法启动原因是5050端口占用，但是，在使用``netstat -nplt| grep 50``查看的时候发现并没有启动50端口的占用，所以，可以最终推断是docker在启动的时候，两次去试图打开5050端口，导致docker无法启动，需要看代码，但是这个古老版本的docker代码已经丢失。待我找到以后，研究研究。

在``/etc/init.d/docker``的docker选项中去掉``-H tcp://127.0.0.1:5050`` 后，运行``service docker restart``正常启动，启动docker以后，发现5050端口没有被占用。

