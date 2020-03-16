---
layout: post
title: "linux笔记: conda安装本地文件"
subtitle: "使用conda安装本地anaconda包"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux
---
## 1. 本地安装anaconda包

anaconda本质是python的包，有很多包，在国内镜像源找不到，需要从外网下载，然后，到内网使用本地安装下载。

conda使用本地包安装命令为：

1. 下载安装包到``~/anaconda3/pkgs/ `` 下
2. cd 到 ``~/anaconda3/pkgs/ `` 目录下
3. 使用命令``conda install --use-local  xxx.tar.bz2`` 安装本地包。

第二节实践从``https://anaconda.org`` 下载r-base包，并使用本地安装。

## 2. 使用本地包安装r-base包

命令如下：

```bash
# curl 'https://anaconda.org/r/r-base/3.6.1/download/linux-64/r-base-3.6.1-h9bb98a2_1.tar.bz2' -Lo ~/anaconda3/pkgs/r-base-3.6.1.tar.bz2
# cd ~/anaconda3/pkgs
# conda install --use-local r-base-3.6.1.tar.bz2
```

运行如上命令，即在本地安装了r-base的anaconda包。


