---
layout: post
title: "k8s笔记：国内安装minikube"
subtitle: "minikube部署单机版本的k8s"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - helm
---

## 前言

由于众所周知的原因，国内安装minikube在下载镜像的时候，会被block住。

查询了网上使用阿里镜像中心下载镜像的minikube的国内改装版，使用体验了一番。

感谢 [https://yq.aliyun.com/articles/221687](https://yq.aliyun.com/articles/221687) 的分享。

## mac环境下实验

1. 安装kubectl：brew install kubectl
2. 安装hyperkit: brew install docker-machine-driver-hyperkit hyperkit
3. 安装minikube
  ```sh
  $ curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
  ```
4. 使用minikube安装单机k8s环境：
  ```
  $ sudo chown root:wheel /usr/local/bin/docker-machine-driver-hyperkit && sudo chmod u+s /usr/local/bin/docker-machine-driver-hyperkit
  $ minikube start --vm-driver=hyperkit --registry-mirror=https://registry.docker-cn.com --kubernetes-version v1.12.1 
  ```
5. 加载minikube中docker的环境变量，使本地的docker命令生效
  ```
  $ minikube docker-env >> ~/.bash_profile
  ```

 6. 其它
   ```
   # 登录到k8s所在机器
   minikube ssh
   # 打开k8s控制台
   minikube dashboard
   ```

## 注意点
1. 本篇文章使用hyperkit作为容器启动的驱动，默认使用virtualbox的驱动，使用不同驱动会有问题

2. 删除hyperkit创建的容器以后，需要删除``～/.minikube``文件夹，删除之前，把``~/.minikube/cache/iso/minikube-v1.2.0.iso``备份出来，删除以后，手动创建~/.minikube目录，并把镜像放到原来位置

3. docker使用端口映射，不能使用本机网络，因为在mac上，docker是起来一个虚拟机来运行docker的，和主机不是同一个主机
