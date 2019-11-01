---
layout: post
title: "docker笔记: 容器中使用systemd"
subtitle: "容器中使用systemctl"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - docker
  - linux
---
## 容器中无法使用systemctl
直接使用``docker run -d --privileged centos init``

## 容器中遇到问题
1. centos没有service服务
centos7 以后去除了service服务，使用systemctl代替
可以使用 ``yum install -y initscripts.x86_64`` 安装service服务

2. centos没有pidof命令
没有pidof的指令集，需要``yum install -y redhat-lsb`` 来对应安装

3. 在centos容器内使用systemctl会出现``Failed to get D-Bus connection: Operation not permitted``
* 方案1
  ```
  docker run -d -e "container=docker" --privileged=true -v /sys/fs/cgroup:/sys/fs/cgroup --name centos7 centos /usr/sbin/init
  ###对安全问题比较关心的同学，建议将--privileged=true替换为--cap-add=SYS_ADMIN
  ```
* 方案2
  ```
  docker run -ti --privileged centos init
  #或者
  docker run -ti --privileged centos /usr/lib/systemd/systemd
  #或者
  docker run -ti --cap-add=SYS_ADMIN cents init
  ```
* 方案3：dockerfile
  ```Dockerfile
  FROM fedora:rawhide
  MAINTAINER “Dan Walsh” <dwalsh@redhat.com>
  ENV container docker
  RUN yum -y update; yum clean all
  RUN yum -y install systemd; yum clean all; 
  (cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i   == systemd-tmpfiles-setup.service ] || rm -f $i; done); 
  rm -f /lib/systemd/system/multi-user.target.wants/*;
  rm -f /etc/systemd/system/*.wants/*;
  rm -f /lib/systemd/system/local-fs.target.wants/*; 
  rm -f /lib/systemd/system/sockets.target.wants/*udev*; 
  rm -f /lib/systemd/system/sockets.target.wants/*initctl*; 
  rm -f /lib/systemd/system/basic.target.wants/*;
  rm -f /lib/systemd/system/anaconda.target.wants/*;
  VOLUME [ “/sys/fs/cgroup” ]
  CMD [“/usr/sbin/init”]
  ```

## reference
* [https://www.cnblogs.com/mar-q/p/7758889.html](https://www.cnblogs.com/mar-q/p/7758889.html)
* [https://developers.redhat.com/blog/2014/05/05/running-systemd-within-docker-container/](https://developers.redhat.com/blog/2014/05/05/running-systemd-within-docker-container/)