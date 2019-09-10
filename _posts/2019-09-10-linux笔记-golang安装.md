---
layout: post
title: "linux笔记：golang安装"
subtitle: "centos环境安装golang"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - golang
  - linux
---

## 安装步骤
1. 下载安装包：
  [https://studygolang.com/dl](https://studygolang.com/dl)
2. 解压到/usr/local：
  ``$ tar -C /usr/local/ -xzvf go1.10.2.linux-amd64.tar.gz``
3. 设置环境变量
  ```sh
  $ vim /etc/profile
	export GOROOT=/usr/local/go
    export GOPATH=/home/zhangboo/goProject 
    export GOBIN=$GOPATH/bin
    export PATH=$PATH:$GOROOT/bin
    export PATH=$PATH:$GOPATH/bin
  ```
4. 使环境变量生效
  ``$ source /etc/profile``

## 参考
[https://www.jianshu.com/p/8f0646e3858c](https://www.jianshu.com/p/8f0646e3858c)
