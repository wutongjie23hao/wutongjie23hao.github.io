---
layout: post
title: "k8s笔记：helm分享"
subtitle: "helm 知识整理"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - helm
---

## 什么是helm

helm是k8s平台的包管理器。就像centos上的yum，nodejs的npm，python的pip。

## 为什么需要helm

在没有helm之前，如果想要在k8s上部署一个应用服务，是一个很复杂的过程。 比如说，部署一个mysql服务，需要有存储数据的pv/pvc，存储配置文件的config map，外界访问到mysql的定义service，还有本身定义的pod的yaml文件，这一系列的k8s资源都要创建出来，比较繁琐。

helm干了一件什么事情呢，把这些资源对应的yaml文件弄成模版，变的部分抽象出来配置文件。使用go 模版语言构建。

## helm架构和概念

### helm概念

将helm之前要知道四个概念：

1. chart ： helm是包管理器，这个包就是chart包。helm的一种打包格式。
2. release ： chart跑起来的一个实例
3. helm client ： 管理chart, helm 组件
4. tiller ： 管理relese, helm 组件

chart/release/helm client/tiller 和docker的东西来类比，表如下：

| -- | -- | -- | -- | -- |
| [docker client](https://github.com/docker/engine)| [docker daemon](https://github.com/docker/engine) | docker image | [docker registry](https://github.com/docker/distribution) | [docker container](https://containerd.io/) |
| [helm client](https://github.com/helm/helm) | tiller | [chart](https://github.com/helm/charts) | [chart repository](https://github.com/helm/chartmuseum) | release |

### helm架构

![helm架构](/img/post/helm.ar.jpg)

1. helm client：终端用户使用的命令行工具，用户可以：
   1. 本地开发 chart
   2. 管理 chart 仓库
   3. 与 Tiller 服务器交互
   4. 在远程 Kubernetes 集群上安装 chart
   5. 查看 release 信息
   6. 升级或卸载已有的 release
2. tiller作用：Tiller 服务器运行在 Kubernetes 集群中，它会处理 Helm 客户端的请求，与 Kubernetes API Server 交互。Tiller 服务器负责：
   1. 监听来自 Helm 客户端的请求
   2. 通过 chart 构建 release
   3. 在 Kubernetes 中安装 chart，并跟踪 release 的状态
   4. 通过 API Server 升级或卸载已有的 release

总结来看：Helm 客户端负责管理 chart；Tiller 服务器负责管理 release。

## helm安装和部署

安装命令：

```sh
# 下载helm安装文件
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get | bash
# 指定k8s的context、tiller的namespace、tiller的image运行tiller
$ alias helm='--kube-context <k8s context> --tiller-namespace <namespace>'
$ helm init --tiller-image <tiller image>
$ helm version
```