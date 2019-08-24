---
layout: post
title: "k8s部署工具"
subtitle: "helm2/helm3/kustomize介绍"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - kubernetes
---
## 前言
> 本文主要介绍下k8s下的包管理器helm，自helm3以后，去除了tiller组件，我比较支持，但是社区反应平平，并且很大部分用户转向kustomize。

## 1. helm 2
helm为k8s平台下的包管理器，官网:[http://www.helm.sh](http://www.helm.sh)，github地址: [https://github.com/helm/helm](https://github.com/helm/helm)，release列表: [https://github.com/helm/helm/releases](https://github.com/helm/helm/releases)。
### 1.1 chart和release
理解helm，需要理解两个概念：chart和release。其中，chart为应用或者服务的模版库，release为chart的一次运行实例或者集群。和os上的程序和进程的概念相似。

helm为k8s平台下的包管理器，所谓的包就是chart。安装chart后的应用或者服务实例就是release。

### 1.2 helm命令
* ``helm init``： 安装tiller到k8s集群，可指定namespace和tiller的镜像
* ``helm reset``：删除tiller
* ``helm repo list``：列出本地的chart 仓库列表
* ``helm repo add <repo_name> <repo_url>``: 增加chart仓库到本地
* ``helm push <chart> <repo>``(需要安装相关插件): push本地chart到指定仓库
* ``helm package <chart>``: 打包本地chart为tgz文件，在安装有requirements.yaml文件的chart时候，有用
* ``helm install <chart>``: 安装指定chart到集群 
* ``helm list``: 将当前k8s集群的release列出来
* ``helm upgrade <release> <chart>``: 用指定chart更新指定release
* ``helm status <release>``: 查看release详情

此外还有回滚、查看指定release历史版本等命令。

### 1.3 参考
[helm github](https://github.com/helm/helm)

## 2. helm 3
helm 3 是helm2的升级版本。升级后，改动比较大。
相对于helm2，helm3除了去除了tiller组件之外，还有以下几方面更新：
1. 可使用镜像仓库进行chart的上传下载。
2. 增加了namespace的权限控制。

参考： [helm release](https://github.com/helm/helm/releases)
## 3. kustomize
kustomize为k8s配置管理工具。

kustomize是在已经有的resource基础上，进行overlay来覆盖写必要变动值的概念。和docker镜像的概念有点相似，有base、patch的概念。

官网:[https://kustomize.io](https://kustomize.io), github地址: [https://github.com/kubernetes-sigs/kustomize](https://github.com/kubernetes-sigs/kustomize)

相对于helm，kustomize有两方面优点：
1. 不需要重新学习go tpl语言
2. 对模版的代码浸入性比较小

### 3.1 使用步骤
1. 创建kustomize文件。
    
    在包含了定义了k8s资源的yaml文件夹中创建名为``kustomization.yaml``的文件。 

    这个文件用来对已经存在的k8s资源文件进行覆写。 
    
    最后的文件结构如下： 
  ![base](/img/post/kustomize.base.jpg) 
  文件结构如下： 
    ```
    ~/someApp
    ├── deployment.yaml
    ├── kustomization.yaml
    └── service.yaml
    ```
    生成yaml文件：``kustomize build ~/someApp`` 

    使用生成的yaml到特点的k8s集群：``kustomize build ~/someApp | kubectl apply -f -``
    > 备注： kustomize不像helm，自己实现了 kubectl一样 的客户端，只是生成yaml文件

2. 继承和patches
  通过继承基础配置来适应不同的发布环境，比如说开发、测试和生产环境。
  ![overlay](/img/post/kustomize.overlay.jpg)
  文件结构：
    ```bash
    ~/someApp
    ├── base
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   └── service.yaml
    └── overlays
        ├── development
        │   ├── cpu_count.yaml
        │   ├── kustomization.yaml
        │   └── replica_count.yaml
        └── production
            ├── cpu_count.yaml
            ├── kustomization.yaml
            └── replica_count.yaml
    ```
    ``base``用来存放各个环境的公共配置
    base中kustomization.yaml内容如下：

    ```yaml
    resources:
    - deployment.yaml
    - service.yaml
    ```

    在overlay中可以自定义子变量，上面定义了两个子环境(development/production):
    overlays/development/kustomization.yaml文件如下：

    ```yaml
    bases:
    - ../../base

    patches:
    - cpu_count.yaml
    - replica_count.yaml
    ```

    overlays下继承了base的配置，并且在此基础上添加了patch：cpu_count.yaml、replica_count.yaml配置，另外，customize不仅支持文件级别的patch，还支持对一个文件某些字段的patch，如下所示replica_count.yaml只包含了有关replicas的部分即可，在执行kustomize build之后，会将这部分覆盖。

    部署使用：
    ```
    kustomize build ~/someApp/overlays/production | kubectl apply -f -
    ```


## 3.2 参考
[kubernetes-使用Kustomize部署应用](https://blog.csdn.net/kozazyh/article/details/89092839)

[kustomize github](https://github.com/kubernetes-sigs/kustomize)

    