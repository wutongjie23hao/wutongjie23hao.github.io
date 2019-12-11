---
layout: post
title: "client-go依赖问题"
subtitle: "编译tekton的dashboard引发问题"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - k8s
  - kubernetes
---
## 问题
在编译tekton和tekton的dashboard的时候，报错client-go的错误。

解决方法：修改``go.mod``中修改 ``k8s.io/api``,``k8s.io/apimachinery``,``k8s.io/client-go``,``k8s.io/sample-controlle``. 修改如下：
```
$ cat go.mod
	k8s.io/api kubernetes-1.15.3
	k8s.io/apimachinery kubernetes-1.15.3
	k8s.io/client-go kubernetes-1.15.3
	k8s.io/sample-controller kubernetes-1.15.3
```
修改``go.mod``以后，重新``go mod vendor``即可。

原因： ``api/apimachinery/client-go``版本之间存在版本兼容问题。