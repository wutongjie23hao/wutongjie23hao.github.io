---
layout: post
title: "docker笔记: 把iso镜像转换成docker镜像"
subtitle: "virtualbox种创建虚拟机，并制作镜像"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker

---

### 1. 先决条件

* iso镜像
* virtual box

启动virtual box，以iso镜像创建虚拟机。

### 2. 在虚拟机中制作docker镜像

1. 生成docker镜像的tar包

   ```bash
   mkdir hello
   tar -cvpf /hello/system.tar --directory=/ --exclude=proc  --exclude=hello --exclude=sys --exclude=dev --exclude=run --exclude=boot .
   ```

2. 通过网络或者文件共享，把system.tar包拷贝到安装有docker的主机上

   ```
   cat system.tar | docker import - system:latest
   ```

> 备注：这个生成的镜像文件比较大，存在一些虚拟机兼容的垃圾文件。官方不推荐这么做。

### 3. 使用mkimage脚本来制作docker镜像

脚本地址：[https://github.com/moby/moby/tree/master/contrib](https://github.com/moby/moby/tree/master/contrib)

以一个centos的为例：

```bash
sh mkimage-yum.sh centos
```


