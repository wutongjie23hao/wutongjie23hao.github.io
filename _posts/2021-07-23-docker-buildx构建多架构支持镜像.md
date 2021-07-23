---
layout: post
title: "docker笔记: docker buildx构建多架构支持镜像并推送至自签名私有镜像仓库"
subtitle: "mac/centos8上构建多架构支持镜像，并推送至自签名的私有镜像仓库"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux


---

## 1. 构建buildkit镜像

### 1.1 dockerfile

```dockerfile
FROM moby/buildkit:buildx-stable-1 
ENV GODEBUG=x509ignoreCN=0
```

### 1.2 构建支持推送至自签名的私有镜像仓库镜像

```shell
docker build --rm -t testbuilder:latest .
```



## 2. 创建buildx的构建实例

```shell
docker buildx create --use --driver-opt image=testbuilder:latest,network=host
```



## 3. mac下尝试构建

### 3.1 多架构镜像的dockerfile

```dockerfile
# syntax=docker/dockerfile:1
FROM --platform=$TARGETPLATFORM golang:alpine AS build
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM" > /log
FROM --platform=$TARGETPLATFORM alpine
COPY --from=build /log /log
```


### 3.2 buildx构建命令

```shell
docker buildx build --no-cache --push --platform linux/arm64/v8,linux/amd64 -t liangxiaolei.fun/myimage -f mydockerfile .
```

报错：

```shell
error: failed to solve: rpc error: code = Unknown desc = failed to do request: Head "https://liangxiaolei.fun/v2/myimage/blobs/sha256:00000000000000000000000000": x509: certificate signed by unknown authority
```

### 3.3 修复

```dockerfile
$ docker ps|grep 'testbuilder'
ee110c9e6dfc   testbuilder:latest       "buildkitd"              27 minutes ago   Up 23 minutes             buildx_buildkit_distracted_payne0

$ docker exec -it ee110c9e6dfc sh
$$ cat >> /etc/ssl/certs/ca-certificates.crt <<'EOF'
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
EOF
$$ exit

$ docker restart ee110c9e6dfc
```

然后进行：

```shell
docker buildx build --no-cache --push --platform linux/arm64/v8,linux/amd64 -t liangxiaolei.fun/myimage -f mydockerfile .
```

可正常push了。



## 4. centos下构建

### 4.1 下载buildx

```shell
wget https://github.com/docker/buildx/releases/download/v0.6.0/buildx-v0.6.0.linux-amd64
chmod +x buildx-v0.6.0.linux-amd64
mv buildx-v0.6.0.linux-amd64 buildx
mv buildx /usr/local/bin
```

### 4.2 构建dockerfile

```dockerfile
FROM --platform=$TARGETPLATFORM golang:alpine  AS build
ARG TARGETPLATFORM
ARG BUILDPLATFORM
RUN echo "I am running on $BUILDPLATFORM, building for $TARGETPLATFORM" > /log
FROM --platform=$TARGETPLATFORM  alpine
COPY --from=build /log /log
```



### 4.3 构建

```shell
buildx build --no-cache --push --platform linux/arm64/v8,linux/amd64 -t liangxiaolei.fun/myimage:tag .
```

错误

```dockerfile
error: failed to solve: rpc error: code = Unknown desc = failed to do request: Head "https://liangxiaolei.fun/v2/myimage/blobs/sha256:00000000000000000000000000": x509: certificate signed by unknown authority
```



### 4.4 修复

```dockerfile
chmod u+w /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem
cat >> /etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem << 'EOF'
---
EOF
```

然后，构建命令为：

```dockerfile
GODEBUG=x509ignoreCN=0 buildx build --no-cache --push --platform linux/arm64/v8,linux/amd64 -t liangxiaolei.fun/myimage:tag .
```



## 5. 参考

* [Custom registry, push error on self-signed cert](https://github.com/docker/buildx/issues/80)
* [docker buildx create](https://docs.docker.com/engine/reference/commandline/buildx_create/)
* [docker buildx](https://docs.docker.com/buildx/working-with-buildx/)


