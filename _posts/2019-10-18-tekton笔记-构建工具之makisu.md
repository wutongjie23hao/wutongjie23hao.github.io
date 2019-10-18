---
layout: post
title: "CI工具makisu使用和推送镜像到harbor"
subtitle: "构建工具之makisu"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - tekton.build
  - k8s
---

## 修改使支持推送到harbor镜像中心
1. [harbor](https://github.com/goharbor/harbor)在收到push的notification的时候，会去扫描请求的user-agent字段，只会把``registry``、``vic/``（还有一个jobservice内部使用的，在这边不做记录）开头的插入到数据库，所以需要对应的修改。
2. 自己搭建的私有镜像中心，使用makisu需要跳过server端的证书认证，相应修改已经提交pr，[https://github.com/uber/makisu/pull/277](https://github.com/uber/makisu/pull/277)，可参考对应修改。

## 编译
```
cd makisu
export GO111MODULE=on
export GOPROXY=https://goproxy.io
go mod download
go mod vendor
docker run --rm -v $(pwd):/go/src/github.com/uber/makisu --workdir /go/src/github.com/uber/makisu golang:1.12.6  make lbins
```

## 封装成docker
1. 新建Dockerfile或者修改默认Dockerfile
```
FROM scratch
COPY bin/makisu/makisu.linux /makisu-internal/makisu
ADD ./assets/cacerts.pem /makisu-internal/certs/cacerts.pem

ENTRYPOINT ["/makisu-internal/makisu"]
```

2. 执行构建镜像的命令
```
docker build --rm -t makisu  .
```

## 使用
```
docker run --rm -v $(pwd):/workspace makisu --modifyfs=true  --push=dockerhub.com --file=/workspace/Dockerfile --tag=dockerhub.com/hello/world:tag build /workspace
```
## makisu‘s github
[https://github.com/uber/makisu](https://github.com/uber/makisu)
