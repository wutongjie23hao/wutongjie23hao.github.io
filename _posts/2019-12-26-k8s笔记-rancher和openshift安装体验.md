---
layout: post
title: "k8s笔记：rancher和openshift安装"
subtitle: "cnetos7下rancer和openshift体验"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - kubernetes
---
# 1、openshift
## 安装
### 前提
[安装docker-ce](http://www.liangxiaolei.fun/2019/11/18/docker%E7%AC%94%E8%AE%B0-centos%E4%B8%AD%E5%AE%89%E8%A3%85docker-ce/)
### 安装
```
# cd /opt/
# curl -Lo openshift-origin-server-v3.6.1.tar.gz  https://github.com/openshift/origin/releases/download/v3.6.1/openshift-origin-server-v3.6.1-008f2d5-linux-64bit.tar.gz
# tar -zxvf openshift-origin-server-v3.6.1.tar.gz
# mv openshift-origin-server-v3.6.1-008f2d5-linux-64bit openshift
# cd /opt/openshift
# yum install net-tools -y
# ifconfig
# ./openshift start --public-master=https://<ip>:8443
```

通过``https://<ip>:8443``访问，用户名/密码：dev/dev
