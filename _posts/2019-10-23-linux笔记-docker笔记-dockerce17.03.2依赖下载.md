---
layout: post
title: "docker笔记：docker ce17.03.2 依赖下载"
subtitle: "go: github.com/Sirupsen/logrus@v1.4.2: parsing go.mod: unexpected module path github.com/sirupsen/logrus"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
---

# 1. docker 依赖下载命令

使用go1.12版本

```
$ go mod vendor
```

# 2. 问题

运行报错：``go: github.com/Sirupsen/logrus@v1.4.2: parsing go.mod: unexpected module path "github.com/sirupsen/logrus"``

# 3. 解决方法

编辑或者新建docker工程目录下面的go.mod文件，内容如下：

```
module github.com/docker/docker

go 1.12

require (
  github.com/Sirupsen/logrus v0.11.0
)

```

运行命令：``go mod download``，会把``Sirupsen/logrus``的依赖下载下来。

然后，再运行命令：``go mod vendor``, 依赖可正常运行。

# 4. 原因
github.com/Sirupsen/logrus 和 github.com/sirupsen/logrus 是两个不同的库。

# 5. 参考
* [https://gist.github.com/myitcv/f7270ab81ab45aa286f496264f034b56](https://gist.github.com/myitcv/f7270ab81ab45aa286f496264f034b56)
* [https://github.com/golang/go/issues/28489](https://github.com/golang/go/issues/28489)
