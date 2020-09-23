---
layout: post
title: "国内centos离线安装kubectl和minikube"
subtitle: "k8s笔记：国内使用minikube安装单机版的k8s"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - kubernetes
---
## 安装docker-ce
[参看之前文档](http://www.liangxiaolei.fun/2019/11/18/docker%E7%AC%94%E8%AE%B0-centos%E4%B8%AD%E5%AE%89%E8%A3%85docker-ce/)

## 离线安装kubectl
```bash
curl -Lo kubectl https://storage.googleapis.com/kubernetes-release/release/$(curl https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x kubectl
mv kubectl /usr/local/bin
ln -sf /usr/local/bin/kubectl /usr/bin/kubectl
```

## 离线安装minikube
```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64&& chmod +x minikube
chmod +x minikube
mv minikube /usr/local/bin/
ln -sf /usr/local/bin/minikube /usr/bin/minikube
```

## 使用minikube安装k8s
```bash
# 安装k8s
minikube start --vm-driver=none --image-repository gcr.azk8s.cn/google-containers
# 安装指定版本的k8s
minikube start --vm-driver=none --image-repository gcr.azk8s.cn/google-containers --kubernetes-version='v1.14.3'
```

## 问题
1. 使用azks.cn的镜像仓库，没有``gcr.azk8s.cn/google-containers/storage-provisioner:v1.8.1``

解决方法：
```bash
docker pull k8sminikube/storage-provisioner:v1.8.1
docker tag k8sminikube/storage-provisioner:v1.8.1 gcr.azk8s.cn/google-containers/storage-provisioner:v1.8.1
```

2. 可用的镜像registry

```
minikube delete
minikube start --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers --kubernetes-version v1.16.2 --vm-driver=none
```
3. 删除并使用新版本k8s
```
# docker 19.03-ce
minikube delete --all
/bin/rm -rf /etc/kubernetes
minikube start --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers --kubernetes-version v1.12.10 --vm-driver=none
```
