---
layout: post
title: "k8s笔记：rancher和openshift安装"
subtitle: "rancer和openshift体验"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - kubernetes
---
# 1、openshift
## 安装
```
# cd /opt/
# curl -Lo openshift-origin-server-v3.6.1.tar.gz  https://github.com/openshift/origin/releases/download/v3.6.1/openshift-origin-server-v3.6.1-008f2d5-linux-64bit.tar.gz
# tar -zxvf openshift-origin-server-v3.6.1.tar.gz
# mv openshift-origin-server-v3.6.1-008f2d5-linux-64bit openshift
# cd /opt/openshift
# yum install net-tools -y
# ifconfig
# ./openshift start --public-master=https://<ip>:443
```