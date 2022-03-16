---
layout: post
title: "golang笔记：cgo_enabled=1构建docker镜像"
subtitle: "cgo_enabled 构建docker镜像注意事项"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - golang
---

## 前言
cgo_enabled=0 的时候，直接二进制文件打进基础镜像，启动就行，和二进制文件在哪里编译生成的没有关系。在cgo_enabled=1的情况下，则要求二进制文件需要在和基础镜像系统一样的宿主机上或者基础镜像上编译生成。

## dockerfile实例
```Dockerfile
FROM centos:7 as base
RUN curl -Lo /tmp/tmp.tar.gz "https://studygolang.com/dl/golang/go1.17.8.linux-amd64.tar.gz" --connect-timeout 10 -m 1200 --insecure &&\
    tar -C /usr/local -xzf /tmp/tmp.tar.gz &&\
    /bin/rm -rf /tmp/tmp.tar.gz ;\
    GODIR=/usr/local/go/bin; for item in $(ls $GODIR); do /bin/rm -rf /bin/$item; ln -s $GODIR/$item /bin/$item; done;\
    history -c
RUN yum install gcc -y 
COPY . /opt
WOKDIR /opt
RUN CGO_ENABLED=1 go build -o BINARY main.go

FROM centos:7
COPY --from=base /opt/BINARY /BINARY
ENTRYPOINT ["/BINARY"]
```

直接`` docker build -t binary . ``即可。
