---
layout: post
title: "CI工具kaniko使用和推送镜像到harbor"
subtitle: "构建工具之kaniko"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - tekton.build
  - k8s
---

## 修改使支持推送到harbor镜像中心
1. [harbor](https://github.com/goharbor/harbor)在收到push的notification的时候，会去扫描请求的user-agent字段，只会把``registry``、``vic/``（还有一个jobservice内部使用的，在这边不做记录）开头的插入到数据库，所以需要对应的修改。
2. kaniko对于以``.local``结尾的非授权镜像中心只走http，不走https，所以几乎不能用，现在相关依赖已经修复，只需拉取最新代码进行编译就行。也可以自行进行相关修改，已经提交pr,[https://github.com/GoogleContainerTools/kaniko/pull/817/files](https://github.com/GoogleContainerTools/kaniko/pull/817/files)，可以对应修改。

## 编译
```
cd kaniko
export GO111MODULE=on
export GOPROXY=https://goproxy.io
go mod download
go mod vendor
docker run --rm -v $(pwd):/go/src/github.com/GoogleContainerTools/kaniko --workdir /go/src/github.com/GoogleContainerTools/kaniko golang:1.12.6  make
```

## 封装成docker
1. 新建Dockerfile或者修改默认Dockerfile
```
FROM scratch
COPY out/executor /kaniko/executor
ENTRYPOINT ["/kaniko/executor"]
```

2. 执行构建镜像的命令
```
docker build --rm -t kaniko  .
```

## 使用
```
docker run --rm -v $(pwd):/workspace kaniko --dockerfile=/workspace/Dockerfile --destination=docker.com/hello/world:tag --context=/workspace --skip-tls-verify-registry=docker.com

```
## kaniko‘s github
[https://github.com/GoogleContainerTools/kaniko](https://github.com/GoogleContainerTools/kaniko)
